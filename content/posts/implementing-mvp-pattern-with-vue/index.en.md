---
title: "Implementing Model-View-Presenter Pattern With Vue"
date: 2021-08-08T23:14:01-04:00
draft: true
tags: ["Vue"]
categories: ["Frontend"]
toc: true
featuredImage: ""
summary: "MVP Pattern with Vue"
---

## Background

For the past few weeks, my major task has been refactoring the old step-wise drafting process of our app into a much more simplified one. Instead of dividing the drafting and submission process into several steps like _Drafting_, _Tagging_, _Feedback_ and _Finalize_, the goal is to have one unified interface where the features previously scattered across different steps can be integrated in one UI by a plug-in manner, similar to how VS Code plugins add new functionalities to the [Activity Bar and Side Bar](https://code.visualstudio.com/docs/getstarted/userinterface).

![New Drafting Process Design](a-new-drafting-process.svg)

The benefits are obvious: No duplicate event handlers implemented for shared components used across different pages; No need to care about updating data to the backend before navigating to other steps; The app feels more like an SPA and less like web pages; The new UI feels much more intuitive to new users, etc.

## The Migration

### Dynamic Component

Based on the requirements, it is pretty clear that we want to load the feature components in a dynamic fashion into the left and right panel. Vue provides support for this OOB via [dynamic components](https://vuejs.org/v2/guide/components.html#Dynamic-Components).

Using dynamic components is easy. You only need to bind the component definition or name to Vue's `<component>` using the `is` attribute:

```html
<component :is="currentRightComponent"></component>
```

Cool! Let's try to dynamically bind the `ExampleView` to a `<component>` in right panel. Wait a minute... What about the props and event bindings?

```Vue
<example-view
  :example="example"
  :visible="true"
  @next="showNextExample"
  @previous="showPreviousExample"
  @save="favoriteExample"
  @focus="hideTag"
/>
```

Lucky for us, all the Vue directives work in the dynamic `<component>` as well. So in a similar fashion, you can also pass props and bind event handlers to a dynamic component like so:

```html
<component
  v-if="currentRightComponent"
  :is="currentRightComponent"
  v-bind="currentRightComponentProps"
  v-on="currentRightComponentEvents"
></component>
```

Here `currentRightComponentProps` and `currentRightComponentEvents` are both local data properties. For a feature component `ExampleView`, these two values could look something like these:

```js
exampleViewComponentProps() {
  const localExample = this.example;
  return {
    example: localExample,
    visible: true,
  };
}
```

```js
exampleViewComponentEvents() {
  return {
      next: this.showNextExample,
      previous: this.showPreviousExample,
      save: this.favoriteExample,
      focus: this.hideTag,
  };
}
```

And the click event handler on the corresponding item within the right sidenav will trigger the dynamic loading:

```Vue
// template
<drafting-side-bar class="sidebar">
  <img
    class="icon"
    src="/static/images/icons/examples.png"
    alt=""
    @click="loadRightComponent('ExampleView')"
  />
</drafting-side-bar>

// script
import ExampleView from "shared/components/example-view/ExampleView.vue";

loadRightComponent(componentName) {
  if (componentName === this.currentRightComponent?.name) {
    this.currentRightComponent = null;
  } else {
    switch (componentName) {
      case "ExampleView":
        this.currentRightComponent = ExampleView;
        this.currentRightComponentProps = this.exampleViewComponentProps;
        this.currentRightComponentEvents = this.exampleViewComponentEvents;
        break;
      default:
        break;
    }
  }
},
```

Nice! We have a functional dynamic component setup and `ExampleView` works as expected! :sparkles: However, we have introduced a big problem here. Have you spotted it? :detective:

That's right! All these `ExampleView`'s implementation details (data, computed properties, event handlers, etc.) are actually living inside the `Drafting` component. Not only does it violate the single responsibility principle, but it also maintainability and extensibility down the road. As more "plugin" features are added to this interface, we can expect a 1000-line Vue component in the not-too-distant future. :scream: We definitely need a better structure to organize this, and that brings us to the meat of this blog post the Model-View-Presenter pattern.

### MVP (Model-View-Presenter) Pattern
