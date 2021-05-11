# vue-2-dropdown

Vue2 plugin for tooltips, popovers and dropdown.
The plugin provides a VDropdown component which is very easy to use.

## Features
- SSR support
- Ability to set alternative placement in cases when the pop-up does not fit into the screen's visibility
- Flexibility and ease of use

## Install
```
npm install --save vue-2-dropdown
```

## Global Registration
```javascript
import Vue from 'vue'
import Dropdown from 'vue-2-dropdown'
import 'vue-2-dropdown/dist/vue-2-dropdown.css';

Vue.use(Dropdown)
```

## Local Registration
```javascript
import { VDropdown } from 'vue-2-dropdown'
import 'vue-2-dropdown/dist/vue-2-dropdown.css';

export default {
  components: {
    VDropdown,
  },
}
```

## Usage
``` html
<template>
  <v-dropdown>
    <div>trigger</div>

    <template #portal>
      <div>content</div>
    </template>
  </v-dropdown>
</template>
```

## Props
Name                | Type                | Default               
--------------------|---------------------|-----------------------
shown               | boolean             | false                 
arrow               | boolean             | true                  
placement           | string              | 'bottom-center'       
trigger             | string              | 'click'               
relativeEl          | string,HTMLElement  | ''                    
viewportEl          | string,HTMLElement  | ''                    
documentTargets     | array               | ['portal', 'trigger'] 
offset              | array               | [0, 0, 0, 0]          
viewportOffset      | array               | [0, 0, 0, 0]          
portalClass         | string,object,array | ''                    
triggerClass        | string,object,array | ''                    
autoHide            | boolean             | true                  
hoverPortal         | boolean             | false                 
clickPortal         | boolean             | false                 
disabled            | boolean             | false                 
noPadding           | boolean             | false                 
zIndex              | string,number       | 50                    
transition          | string              | 'fade'                
transitionMode      | string              | 'in-out'              
mountTo             | string              | 'body'                
mountSelf           | boolean             | false                 

### shown
The parameter is responsible for displaying the pop-up. It can be useful when more complex logic for displaying a popup is needed. **Supports sync modifier**. In this case, you will most likely have to configure the following options: *trigger*, *documentTargets*

### arrow
Arrow display

### placement
A very important parameter. Has the following meanings:
- "top-start"
- "top-center"
- "top-end"
- "top-stretch"
- "right-start"
- "right-center"
- "right-end"
- "right-stretch"
- "bottom-start"
- "bottom-center"
- "bottom-end"
- "bottom-stretch"
- "left-start"
- "left-center"
- "left-end"
- "left-stretch"

The first word before the hyphen specifies the position relative to the trigger, the second indicates the alignment relative to the trigger. **These values can be combined: 'bottom-center|top-center'** - this will mean that the pop-up will be displayed in the bottom center, but if it does not fit into the viewport of the window, then it will move to the top center.

## trigger
Trigger at which the pop-up will open. Possible values: *'click', 'hover', ''*. An empty string is indicated if you do not need triggers and want to do the logic yourself using the **shown** parameter

## relativeEl
HTMLElement relative to which the popup will be positioned.

``` html
<template>
  <div>
    <div ref="relativeEl">relative el</div>
    <v-dropdown :relative-el="$refs.relativeEl">
      <div>trigger</div>

      <template #portal>
        <div>content</div>
      </template>
    </v-dropdown>
  </div>
</template>
```

## viewportEl
The element from which to read the viewport. Best used with **mountSelf** parameter

``` html
<template>
  <div ref="viewportEl" style="overflow: scroll;">
    <div>relative el</div>
    <v-dropdown mount-self :viewport-el="$refs.viewportEl">
      <div>trigger</div>

      <template #portal>
        <div>content</div>
      </template>
    </v-dropdown>
  </div>
</template>
```

## documentTargets
This option is only meaningful if the *autoHide* parameter is enabled.
By default, the pop-up will close when clicking on any element except the *trigger* and the *portal*. They can be changed:

``` html
<template>
  <div>
    <v-dropdown :document-targets="['portal']">
      ...
    </v-dropdown>
  </div>
</template>
```
then the pop-up will close when you click on any element other than the *portal*.

## offset
Pop-up offset. The first element of the array sets the offset along the x-axis, the second along the y-axis. the third and fourth are needed to offset the alternative placement (the third is x, the fourth is y).

## viewportOffset
Sets the offset of the visible area of the window. ([top, right, bottom, left]).

## portalClass
css classes of portal.

## triggerClass
css classes of trigger.

## autoHide
Closes the popup when clicking on any element except: **documentTargets**.

## hoverPortal
Only meaningful when trigger = 'hover'. Popup does not close when hovering over portal.

## clickPortal
Only meaningful when trigger = 'click'. Closes the popup when the portal is clicked.

## disabled
Prevents the pop-up from opening.

## noPadding
Removes paddings from the portal.

## zIndex
z-index of the portal

## transition
css class for transition. There is only *fade* by default. If you want other transitions, you will have to write them yourself.

## transitionMode
transition-mode

## mountTo
Element selector to which the portal will be inserted.

## mountSelf
Prevents the creation of a portal.
