---
title: "Create Vue Components as, not inside, Modals"
date: 2019-12-18T11:07:28-05:00
draft: false
---

Recently, the majority of my work involves integrating Vue in a traditional Django application, which uses jQuery extensively. The first task on this front is to convert a preview modal into a Vue component and add some quick editing functionality. In this post, I will share some refactoring experience and lessons learned along the way.

### The Old Way

The original workflow works as follows:

- In `content-list.html`, there is a list of content cards that contains some content info and action buttons for preview and editing.

```html
<ul>
  {% for obj in object_list %}
  <li class="default-card">
    <div class="card-header">Card Header</div>
    <div class="card-action">
      <button
        class="show-modal"
        data-url="{% url 'preview_content' obj_id=obj.id %}"
      >
        Preview
      </button>
      <button
        class="show-modal"
        data-url="{% url 'edit_content' obj_id=obj.id %}"
      >
        Edit
      </button>
    </div>
    <div class="content-body">Content Body</div>
  </li>
  {% endfor %}
</ul>
```

- In a global JavaScript file `project-script.js`, there are jQuery functions that sets the event listeners for buttons with class `show-modal`.

```js
$(document).ready(function() {
  $(document).on("click", ".show-modal", showClickedModal);
});
```

- Also in that `project-script.js`, there are the implementations for these function:

```js
var modalIsOpen = false;

function showClickedModal(callback) {
  buildModal($(this), callback);
}

function buildModal(elem, callback) {
  var me = $(elem);
  var modalUrl = me.data("src");
  modalConstructor(modalUrl, callback);
}

function modalConstructor(modalUrl, callback) {
  var html = '<div class="modal"></div><div class="backdrop"></div>';
  $("body").append(html);
  $("body").on("click", ".backdrop", closeModal);
  $(".modal").html(getLoadingIndicator());
  $.ajax({
    url: modalUrl,
    success: function(data) {
      $(".modal").replaceWith(data);
      $(".modal")
        .children()
        .find("[autofocus]")
        .focus();
      modalIsOpen = true;
      if (typeof callback === "function") {
        callback();
      }
    },
    error: function(data) {
      console.log(data.ResponseText);
    }
  });
}

function getLoadingIndicator() {
  return `<div class="loading-indicator">Loading</div>`;
}

function closeModal() {
  $(".backdrop")
    .add(".modal")
    .remove();
  modalIsOpen = false;
}
```

I know you must be thinking "WOW this piece of code really needs some refactoring!" Well, that's what we are doing, isn't it? Legacy code that got shared between projects and went through the hands of a dozen developers tend to be messy and we should show some love. Let's walk through this snippet of code and see what it's doing:

- `showClickedModal` takes a parameter `callback` and pass it to `buildModal`. This is for cases where you have extra functions that need to be invoked upon creating the modal. Example: `showClickedModal.call(this, initMultiSelect)`.
- `buildModal` grabs the `data-url` attribute from the button and pass it together with the `callback` to `modalConstructor`.
- `modalConstructor` constructs the modal by first creating an HTML element `<div class="modal"></div><div class="backdrop"></div>`, appending it the the end of the page's `<body>` section, and add event listeners on the backdrop area for closing modal. Note that in our case, **all the styles that define the look and feel of a modal come with `class="modal"`.**
- `modalConstructor` then issues an AJAX call to the URL endpoint we just grabbed from `data-url` and **replaces** the placeholder of `<div class="modal"></div>` with the response HTML from that URL. This means Before it receives the response, a loading indicator will be placed in the modal as a placeholder.

This method of constructing modal is good for reusability: You don't need to worry about what is inside the modal. What you have to remember is just to specify the `data-url` attribute for the modal to grab the content (HTML pages). If an additional action needs to be done upon modal creation, you can pass it as the `callback` argument like this: `showModal.call($(this), callbackFunction)`.

However, this is a bit of an overkill if you application only has some modals. It also hinders the Vue-ification of the application. What if you want to build the `Preview` function as a Vue component?

### First Attempt

Let's say we have built a Vue component for the preview modal called `ContentPreview.vue` and we have registered it with Vue. How are we going to load it? Remember that we are using `replaceWith` in the jQuery version of `modalConstructor`, which means that the template loaded from the `data-url` is should have a top level wrapper `div` with `class="modal"`. So one (naive) way we can do it is to leave this top level `div` intact and create a `div` inside for hosting the Vue component.

Previously:

- `content-preview.html`:

```html
<div class="modal">
  <div class="modal-header">{{ content.title }}</div>
  <div class="modal-body">{{ content.text }}</div>
  <div class="modal-footer">{{ content.extra_info }}</div>
</div>
```

After Vue-ification:

- `content-preview.html`:

```html
<div class="modal">
  <div id="content-id">{{ content_id }}</div>
  <div id="content-preview"></div>
</div>

<script>
  if (document.getElementById("content-preview")) {
    let contentId = document.getElementById("content-id").textContent;
    let vueContainer = new Vue({
      render: function(createElement) {
        return createElement("contentPreview", {
          props: {
            contentId: parseInt(contentId)
          }
        });
      },
      el: "#content-preview",
      store
    });
  }
</script>
```

- `ContentPreview.vue`:

```html
<template>
  <div class="modal-header">{{ content.title }}</div>
  <div class="modal-body">{{ content.body }}</div>
  <div class="modal-footer">{{ content.extraInfo }}</div>
</template>

<script>
  import { dataService } from "../data";

  export default {
    name: "ContentPreview",
    props: {
      contentId: Number
    },
    data() {
      return {
        content: {}
      };
    },
    async created() {
      this.content = await dataService.getPreviewContent(contentId);
    }
  };
</script>
```

In this solution, we will only pass the `id` of the content to the Vue constructor and let it handle the HTTP stuff. Of course, we will need to construct the corresponding API endpoint on the backend as well.

However, there are a few problems.

The more obvious one is that, in order to display a modal, we now need to issue two HTTP requests: one for the `content-preview.html` and one for the actual content. This is not clean but does not lead to "bad" consequences.

The more serious problem is that now when we click on the backdrop area, the modal will still be closed as instructed by the `closeModal` function but the Vue instance will not get _destroyed_, meaning that life cycle hooks like `beforeDestroy` and `destroyed` will not be triggered. If the modal is only displaying static content, it's not a big deal. However, if we have a toggle in the preview modal for toggling the active/inactive state of the content, and we want to save the status when we click on the backdrop area, we will not be able to do so.

Clearly, we need some refactoring.

### Improved Solution

How should we solve the second problem? When we come to think more closely about it, the root of this problem is the discrepancy of JavaScript events: `closeModal` is triggered by jQuery while the **save** action (should be defined in `beforeDestroy` life cycle hook) is triggered in a Vue component. What if we include the `backdrop` area in the `ContentPreview.vue`? How about including the `modal` part as well? Why don't we build the Vue component as a modal?

This is highly doable or rather, just intuitive. Remember the composition of a Vue single file component: template, script and style. If we have a template in Vue already, we do not need to load it into another template just to display it. Also remember that a modal is essentially a `div`, and all the styles that make it look like a modal come with `class="modal"`.

With such consideration, we can now refactor the solution like this:

- Remove all jQuery functions for constructing modals and adding event listeners
- Remove `content-preview.html`
- Refactor `content-list.html`:

```html
<ul>
  {% for obj in object_list %}
  <li class="default-card">
    <div class="card-header">Card Header</div>
    <div class="card-action">
      <!-- New content-id attribute -->
      <button class="show-content-preview" content-id="{{ obj.id }}">
        Preview
      </button>
      <button
        class="show-modal"
        data-url="{% url 'edit_content' obj_id=obj.id %}"
      >
        Edit
      </button>
    </div>
    <div class="content-body">Content Body</div>
  </li>
  {% endfor %}
</ul>

<!-- New -->
<div id="content-preview"></div>

<script
  type="text/javascript"
  src="{% static 'js/vue/loaders/content-preview.js' %}"
></script>
```

As shown above, we are placing the modal in `content-list.html` instead of the end of `body` because we know for sure that this `ContentPreview.vue` component will only be used here in conjunction with the preview buttons.

- Add new file `content-preview.js`

```javascript
document.addEventListener(
  "click",
  function(event) {
    if (event.target.matches(".show-content-preview")) {
      showContentPreview.call(event.target);
    }
  },
  false
);

function showContentPreview() {
  if (document.getElementById("content-preview")) {
    let contentId = this.getAttribute("content-id");
    let vueContainer = new Vue({
      render: function(createElement) {
        return createElement("ContentPreview", {
          props: {
            contentId: parseInt(contentId)
          }
        });
      },
      el: "#content-preview",
      store
    });
  }
}
```

- Refactor `ContentPreview.vue`:

```html
<template>
  <div v-if="showModal">
    <div class="modal">
      <div class="modal-header">{{ content.title }}</div>
      <div class="modal-body">{{ content.body }}</div>
      <div class="modal-footer">{{ content.extraInfo }}</div>
    </div>
    <div class="backdrop" @click="closeAndSubmit"></div>
  </div>
  <div v-else id="content-preview"></div>
</template>

<script>
  import { dataService } from "../data";

  export default {
    name: "ContentPreview",
    props: {
      contentId: Number
    },
    data() {
      return {
        content: {},
        showModal: true
      };
    },
    methods: {
      async closeAndSubmit() {
        this.showModal = false;
        await this.dataService.postUpdatedContent(this.content);
      }
    },
    async created() {
      this.content = await dataService.getPreviewContent(contentId);
    }
  };
</script>
```

In the refactored `ContentPreview.vue`, we now have new variable `showModal` that controls whether the modal will be shown. When the component first gets created, `showModal` defaults to `true`. When you click on the backdrop area, method `closeAndSubmit` will be triggered, `showModal` will be set to `false` and updated content will be saved. When `showModal` is `false`, `<div id="content-preview"></div>` will be placed in the HTML instead, **which is exactly the same as before the Vue component gets loaded**. So when the user clicks another preview button, the `showContentPreview` function in `content-preview.js` will be able to find this `div` and mount another new Vue instance (show anther modal).

The benefits of this solution is obvious: You now only need to issue a single HTTP request to show the modal; The backdrop area is a part of the Vue component now so you can add event listeners to it.

### Summary

In this post, we went through two steps of refactoring: from jQuery to Vue and from "Vue in a modal" to "Vue as a modal". We also discovered how we can use `v-if`, `v-else` and the boolean variable `showModal` to control the display of the modal. Hope this post can be helpful in a some way!

Happy coding!
