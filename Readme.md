# Accordion Menu <!-- omit in toc -->

Display the dropdown menu control as an accordion menu. This module works when your menu display direction is set to "TopToBottom" and when the menu is not very deep. 

![](images/view.gif)

# Version
1.0

1.0.1 Various CSS adjustments and fixes

1.1 The main CSS file no longer required in the *head* property of the application. The CSS is now loaded by the global script. All customisation variables commented out by default. 

# Setup

## Application Setup
1. Check the *Enable Style Sheet* checkbox in the application properties

## Global Script
1. Create a Global Script called "AccordionMenu"
2. Add the input parameters below to the Global Script
   1. MenuClass
   2. OpenMultiple
   3. OpenParent
3. Drag a *JavaScript* action into the script
4. Add the Javascript below into the JavaScript code property
```javascript
/* Stadium Script v1.1 https://github.com/stadium-software/accordion-menu */
loadCSS();
let scope = this;
let openMultiple = ~.Parameters.Input.OpenMultiple;
if (openMultiple !== true && openMultiple !== "true") {
    openMultiple = false;
}
let showSelected = ~.Parameters.Input.OpenParent;
if (showSelected !== false && showSelected !== "false") {
    showSelected = true;
}
let menuClass = ~.Parameters.Input.MenuClass;
let menuEl = document.querySelectorAll("." + menuClass + ".menu-container");
if (menuEl.length == 0) {
    console.error("No menu found with class: " + menuClass);
} else if (menuEl.length > 1) {
    console.error("Multiple menus found with class: '" + menuClass + "'. Please use a unique class for each menu.");
}
menuEl = menuEl[0];
menuEl.classList.add("stadium-accordion-menu");
let menus = menuEl.querySelectorAll(".dropdown > a, .dropdown-submenu > a");
for (let i=0;i<menus.length;i++) {
    createMenuToggleEvent(menus[i]);
}
if (showSelected) openSelected();
function createMenuToggleEvent(menu) {
	menu.addEventListener("click", function(e) {
        let target = e.target;
        let menu = target.closest(".dropdown, .dropdown-submenu");
        if (menu.classList.contains("dropdown") && !menu.classList.contains("show") && !openMultiple) {
            closeAllMenus(menu);
        }
        if (menu.classList.contains("dropdown-submenu") && !menu.classList.contains("show") && !openMultiple) {
            closeSiblings(menu);
        }
        if (menu) menu.classList.toggle("show");
        if (!menu.classList.contains("show")) closeSubmenus(menu);
    });
}
function closeSiblings(menu) {
    let siblingmenus = menu.closest(".dropdown-menu").querySelectorAll(".dropdown-submenu");
    for (let i=0;i<siblingmenus.length;i++) {
        if (menu === siblingmenus[i]) continue;
        siblingmenus[i].classList.remove("show");
    }
}
function closeSubmenus(parentMenu) {
    let submenus = parentMenu.querySelectorAll(".dropdown-submenu");
    for (let i=0;i<submenus.length;i++) {
        submenus[i].classList.remove("show");
    }
}
function closeAllMenus() {
    let menus = menuEl.querySelectorAll(".dropdown, .dropdown-submenu");
    for (let i=0;i<menus.length;i++) {
        menus[i].classList.remove("show");
    }
}
function openSelected() {
    let arrPageName = window.location.pathname.split("/");
    let pageName = arrPageName[arrPageName.length - 1];
    let array = dmGet(menuEl, "Items");
    let findItemNested = (arr, itemId, nestingKey) => (
      arr.reduce((a, item) => {
        if (a) return a;
        if (item.url === itemId) return item;
        if (item[nestingKey]) return findItemNested(item[nestingKey], itemId, nestingKey);
      }, null)
    );
    let ob = findItemNested(array, "/" + pageName, "items");
    let menuitemtext = menuEl.querySelectorAll("li > a > span");
      for (let i=0;i<menuitemtext.length;i++) {
          if (menuitemtext[i].textContent === ob.text) {
            let elem = menuitemtext[i].closest("li");
            for (; elem && elem !== document; elem = elem.parentNode) {
                if (elem.classList.contains("dropdown") || elem.classList.contains("dropdown-submenu")) elem.classList.add("show");
            }
            break;
        }
    }
}
function dmGet(control, p){
    let getObjectName = (obj) => {
        let objname = obj.id.replace("-container","");
        do {
            let arrNameParts = objname.split(/_(.*)/s);
            objname = arrNameParts[1];
        } while ((objname.match(/_/g) || []).length > 0 && !scope[`${objname}Classes`]);
        return objname;
    };
    function getDMValues(ob, property) {
        let obname = getObjectName(ob);
        return scope[`${obname}${property}`];
    }
    return getDMValues(control, p);
}
function loadCSS() {
    if (!document.getElementById("stadium-accordion-menu-css")) {
        let cssMain = document.createElement("style");
        cssMain.id = "stadium-accordion-menu-css";
        cssMain.type = "text/css";
        cssMain.textContent = `.container:not(.mobile) .stadium-accordion-menu{interpolate-size:allow-keywords;.dropdown,.dropdown-menu,.dropdown-submenu{border:0;}.navbar-left>li:not(:last-of-type),li.dropdown-submenu:not(:last-of-type){border-bottom:var(--accordion-menu-item-border-bottom-width,0.1rem) solid var(--accordion-menu-item-border-bottom-color,var(--MOBILE-MENU-ITEMS-BOTTOM-BORDER-COLOR,transparent));}.dropdown-menu{float:none;position:static;box-shadow:none;border-left:var(--accordion-submenu-border-width,0.1rem) solid var(--accordion-submenu-border-color,transparent);margin-left:var(--accordion-submenu-margin-right,1.2rem);.dropdown-item-text{width:100%;}}.dropdown.show>a .caret:after,.dropdown.show:hover>a .caret:after,.dropdown-submenu.show>a:first-child:after,.dropdown-submenu.show:hover>a:first-child:after{content:"\\f0d7";}a{display:flex;text-decoration:none;span:nth-child(2){margin-left:auto;gap:0.6rem;padding-right:0.7rem;}}.navbar-left .caret::after,.dropdown-submenu.expand-right>a:after{margin-left:1.2rem;cursor:pointer;color:var(--MENU-ARROW-COLOR,var(--MENU-ITEM-FONT-COLOR));}.navbar-nav>li:hover,.dropdown-menu>li:hover{color:inherit;background-color:inherit;}.dropdown>.dropdown-menu,.dropdown-submenu>.dropdown-menu{display:block;height:0;transition:height var(--accordion-menu-expand-speed,0.35s) ease;overflow-y:clip;padding-bottom:0;padding-top:0;margin-top:0;}.dropdown.show>.dropdown-menu,.dropdown-submenu.show>.dropdown-menu{display:block;height:calc-size(fit-content,size);}.dropdown:hover>.dropdown-menu,.dropdown-submenu:hover>.dropdown-menu{display:inherit;}}.stadium-accordion-menu{a:hover{color:var(--MENU-ITEM-HOVER-FONT-COLOR);background-color:var(--MENU-ITEM-HOVER-BACKGROUND-COLOR);}}/*DO NOT CHANGE BELOW THIS LINE*/html{min-height:100%;font-size:62.5%;.checkbox label,.checkbox-inline label,.radio label,.radio-inline label{font-size:var(--FORM-FONT-SIZE);}}`;
        document.head.appendChild(cssMain);
    }   
}
```

## Page
1. Add a menu control to your page and set the *Direction* property to "TopToBottom"
2. Add a unique class to the menu control *classes* property (e.g. stadium-accordion-menu)

![](images/MenuProps.png)

## Page.Load
1. Drag the global script called "AccordionMenu" into the *Page.Load* event
2. Add the input parameters below to the *Page.Load* event
   1. MenuClass: The unique class you added to the menu control (e.g. stadium-accordion-menu)
   2. OpenMultiple: Set to true if you want to allow opening multiple menus at once (default is false)
   3. OpenParent: Set to false if you don't want the parent menu of the current page to be opened when the page loads (default is true)

## Display Customisations
The module allows for some customisations via CSS variables. The variables are commented out by default, so you can uncomment and adjust them as needed.

1. Open the CSS file called [*accordion-menu-variables.css*](accordion-menu-variables.css) from this repo
2. Uncomment and adjust the variables in the *:root* element as you see fit
3. Add the [*accordion-menu-variables.css*](accordion-menu-variables.css) to the "CSS" folder in the EmbeddedFiles (overwrite)
4. Paste the link tag below into the *head* property of your application (if you don't already have it there)
```html
<link rel="stylesheet" href="{EmbeddedFiles}/CSS/accordion-menu-variables.css">
```

**NOTE: Do not add or remove variables from the 'accordion-menu-variables.css' file**

## Upgrading Stadium Repos
Stadium Repos are not static. They change as additional features are added and bugs are fixed. Using the right method to work with Stadium Repos allows for upgrading them in a controlled manner. 

How to use and update application repos is described here: [Working with Stadium Repos](https://github.com/stadium-software/samples-upgrading)