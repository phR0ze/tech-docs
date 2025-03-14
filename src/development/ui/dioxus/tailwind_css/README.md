# Tailwind CSS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

A utility-first CSS framework packed with utility type classes that can be composed to build any 
design with infinite flexibility. Nothing is pre-styled; not even headings or links. You have to 
create everything from scratch, giving you the opportunity to create something unique. Typically a 
designer will create sets of semantic classes that group the appropriate utiltiy classes for 
component types e.g. `btn`, `btn-primary` etc... and use those everywhere and don't use the utility 
classes directly. 

`Tailwind UI` though is a higher level set of pre-styled components such as `hero`, `sections`, `CTA` 
etc... more like Bulma or Bootstrap but are based on the utility classes of Tailwind CSS. Notably 
though Tailwind UI is not free. 

However there are a host of others like `daisyUI` or `Flowbite` that are built on top of Tailwind CSS 
providing pre-styled components similar to Tailwind UI for free.

### Quick links
- [.. up dir](../README.md)
* [Overview](#overview)
* [Tailwind Reset](#tailwind-reset)

## Overview
Latest version 3.3.2 when I reviewed this first.

**References**
* [Tailwind docs](https://tailwindcss.com/docs/installation)
* [Tailwind showcase](https://tailwindcss.com/showcase)
* [Tailwind CSS Un-reset](https://dev.to/swyx/how-and-why-to-un-reset-tailwind-s-css-reset-46c5)
* [daislyUI](https://daisyui.com/components/alert/)
* [Flowbite components](https://flowbite.com/docs/components/alerts/)
* [Versoly UI](https://versoly.com/versoly-ui/components/alert)
* [Tailwind Stamps](https://tailwindcss.5balloons.info/components/alerts/)
* [Tailwind Elements](https://tailwind-elements.com/docs/standard/components/alerts/)
* [Tailblocks](https://tailblocks.cc/)
* [Tailwind templates](https://tailwindtemplates.co/)
* [Hyper UI](https://www.hyperui.dev/components/application-ui/alerts)
* [Tailwind Components](https://tailwindcomponents.com/)
* [Meraki UI](https://merakiui.com/components)
* [Tailwind Toolbox](https://www.tailwindtoolbox.com/)
* [Tailwind CSS releases](https://github.com/tailwindlabs/tailwindcss/releases)

### Tailwind Reset
Tailwind CSS comes with a great CSS Reset, called Preflight. It starts with the awesome Normalize.css 
project then nukes all default margins, styling, and borders for ever HTML element. This is so that 
you have a consistent, predictable starting point with which to apply your visual utility classes 
separate from the semantic element names. Preflight is included as part of the `base` tailwind 
module.

## Tailwind in Dioxus

### Include Tailwind
Tailwind provides the ability to only include what you use. However to do this you'll need to have a 
tailwind minify operation run during build time to scan your codebase to determine what is being used 
to generate the final `tailwind.min.css`.

**tailwindcss as alternate to npx**
You can install `tailwindcss` and use it instead of executing `npx` commands but it gives you 
the exact same options and is of no benefit.

* [Setup tailwind for yew](https://dev.to/arctic_hen7/how-to-set-up-tailwind-css-with-yew-and-trunk-il9)
* [Optimizing for Production](https://tailwindcss.com/docs/optimizing-for-production)
* [Tailwindcss docs - Installation](https://tailwindcss.com/docs/installation)

1. Initialize TailwindCSS to create `tailwind.config.js`
   ```shell
   $ cd ~/Projects/<project>
   $ npx tailwindcss init
   ```
2. Update `tailwind.config.js`. Any missing sections will fall back to the default configuration
   * [Default Tailwind Config](https://github.com/tailwindlabs/tailwindcss/blob/master/stubs/config.full.js)
   * Use `npx tailwindcss init --full` to get a full default config
   ```javascript
   module.exports = {
     content: ["./src/**/*.{html,rs}"],
     theme: {
       extend: {},
     },
     plugins: [],
   } 
   ```
3. Create module call out `tailwind.input.css`
   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   ```
4. Run the minify operation manually to generate `tailwind.min.css` for your project to start from
  ```shell
  $ npx tailwindcss -c tailwind.config.js -i tailwind.input.css -o tailwind.min.css --minify
  ```
5. Automate the building of it in your project, create `build.rs`
   ```rust
   use std::process::Command;
   
   fn main() {
       Command::new("npx")
           .arg("tailwindcss")
           .arg("-c")
           .arg("./tailwind.config.js")
           .arg("-i")
           .arg("./tailwind.input.css")
           .arg("-o")
           .arg("./tailwind.min.css")
           .arg("--minify")
           .output()
           .expect("Error running tailwindcss");
   }
   ```
6. Include in your project
   ```rust
   pub fn get_tailwind_css() -> &'static str {
       include_str!("../tailwind.min.css")
   }

   fn App(cx: Scope) -> Element {
       cx.render(rsx! {
           style: { "{get_tailwind_css()}" },
           div {
               class: "w-full h-screen bg-gray-300 flex items-center justify-center",
               "Hello, world!"
           }
       })
   }
   ```


