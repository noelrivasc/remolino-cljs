
## About Remolino

Remolino is a tiny library (really, 29 lines) that helps you keep Tailwind styles independent from the hiccup components they are applied to.

This lets me:

- Focus on structure and data (subs, events) when writing components.
- Focus on visual aspects when writing Tailwind styles.
- Keep components readable, without a myriad classes getting in the way.

**NOTE that this library was found to be a convoluted way to do what you can achieve with Tailwind alone.** See [Solving with Tailwind alone](#solving-with-tailwind-alone) below.

## The problem

Say, you have a component written in hiccup. For example:

```clojure
(defn book-card [book]
 [:div.book-card
  [:div.book-card__picture [:img {:src (:cover-image book)}]
  [:h2.book-card__title (:title book)]]])
```
Then, you want to add Tailwind classes to style it. That's easy (but ugly):

```clojure
(defn book-card [book]
 [:div.book-card {:class ["shadow-2xl"
                         "min-h-20"
                         "mx-auto"
                         "rounded-sm"
                         "border"
                         "border-slate-300"
                         "bg-slate-100"]}
  [:div.book-card__picture [:img {:src (:cover-image book)}]
  [:h2.book-card__title (:title book)]]])
```

You can see how adding just a few classes to most of the tags can get hard to read and ugly. This way of adding styles can also lead to dirty diffs and it's just distracting.

- When I write hiccup, I want to focus on functional aspects —markup, data, subs, events.
- When I apply styles, I want to focus on visual aspects.
- I like clean commit diffs whose intention is evident.

## Remolino's solution

**(See [Solving with Tailwind alone](#solving-with-tailwind-alone) for a simpler way.)**

Decouple styles from markup structure by using BEM, a theme map, and a macro to combine them.

### 1. Write hiccup using the BEM naming convention. 

[BEM stands for block element modifier](https://getbem.com/introduction/). BEM proposes using classes to name

- Blocks (what I'd call a component): `.book-card`
- Elements that make those components: `.book-card__picture`
- Variants for any element: `.book-card--highlighted` or `.button--red`

The reason I like BEM is that the class names convey the structure and purpose of the markup, with little ambiguity.

### 2. Write a theme map

A Remolino theme map is a container that tells us what classes to apply to which markup elements.

```clojure
(def colorful-theme {
  ; Top-level keywords match the top-level tag of the
  ; component they are meant to theme
  :div.book-card {
    ; For now, the full tag is used (note the :div is inclucded)
    ; Using classes and allowing composition is planned but not yet
    ; implemented
    :div.book-card ["shadow-2xl"
                    "min-h-20"
                    "mx-auto"
                    "rounded-sm"
                    "border"
                    "border-slate-300"
                    "bg-slate-100"]
    :div.book-card__picture ["rounded-sm"
                             "border"
                             "border-slate-800"
                             "...more classes"]
  }
})
```

### 3. Make a theming macro

Combine your theme with the functions provided by Remolino to make a macro that transforms your hiccup into themed hiccup:

```clojure
(ns my-app.theming-config
  (:require
   [remolino :as r]
   [my-app.themes.slate :as slate]))

(defmacro theme [component] 
 (if (vector? component)
   (r/apply-theme component slate/theme)
   component))
```

### 4. Apply the theme to your components

```clojure
(ns my-app.views
  (:require [my-app.theming-config :refer [theme]]))

(defn book-card [book]
 (theme [:div.book-card
         [:div.book-card__picture [:img {:src (:cover-image book)}]
          [:h2.book-card__title (:title book)]]]))
```

Note that the macro will run at compile time —not at run time. This means that the body of the component (`book-card` in the example above) will be the same that you would have had you applied the classes to the components directly.

## Using with ClojureScript

Using macros in ClojureScript can be tricky —as I recently discovered. The thing is that there are two compilation steps: one that can run Clojure code and things like macros, and another step that compiles to JS. Any transformations to your Clojure must occur in the first step.

While I can't explain the internals of this process, this also means that the kinds of files you're allowed to require is limited by the compilation step in which they'll be processed.

What I've found is that:

- You can't require from .cljs files in .clj.
- You can require from .clj files in .cljs but you need an intermediate step.
- You can require from .cljc files in .cljs.

So, the chain of requires (the one that worked for me, at least) is as follows:

- Remolino library is a .cljc file.
- You write your macro in a .clj file.
- You create a .cljc file to bring your macro to the .cljs world.
- You then require the macro from the file that contains your components.

## Solving with Tailwind alone

Thank you @cjohansen for pointing out a simpler path.

While the documentation for Tailwind 4.x removed the use of `@apply` from the examples in _avoiding duplication_, the `@apply` directive is still there. This allows you to compose new utility classes out of existing ones.

Say you have a component, like the one we've been using as example:

```clojure
(defn book-card [book]
 [:div.book-card
  [:div.book-card__picture [:img {:src (:cover-image book)}]
  [:h2.book-card__title (:title book)]]])
```

Then, in CSS, you can do:

```CSS
@import "tailwindcss";

@utility book-card__title {
  @apply text-xl bg-violet-700 border border-red-700;
}
```

This is the composition that you can achieve with Remolino, but with less moving parts. So unless there's a good reason, consider the library dead. We remain thankful for the more than 10,800 seconds of service it provided to a single user.

<small>

```monospace
               @@@                                
          @@@      @@@@@@@@@@@@@@@@@@@@@          
      @@@@   @@@@@@@@                @@@@@@@@     
    @@@@ -@@@@@                          @@@@@@   
   @@@ @@@@@    @@@@@@@@@@@@@@@@@@@@#      @@@@@@ 
  @@@@@@@@@ @@@@@                   @@@@@  @@@@@@=
  @@@@@@@@@@@                            @@@@@.@@:
  @@@@@@@@@       @@@@@@@@@@@@@@@@@@@@ @@@@@@@@@@ 
    @@@@@@@@@@.@@@@                @@@@@@@  @@@@  
      @@@@@@@@@@@@@@@@@*#@@@@@@@@@@@@@:  -@@@@    
           @@@@@@@@@@@@@@@@@@@@@@     %@@@@:      
        :                         @@@@@           
           @@            @@@@@            @       
              @@@@@@@             +@@@@@@         
                   @@@@@@@@@@@@@@@@@@@            
            @@@.        @@@@@@@@@@@               
              .@@@@@@@@@@@@@@@@                   
                               @                  
             @@@@        @@@@@                    
               @@@@@@@@@@@                        
                                                  
                 @@@@@@@@                         
               @%                                 
               @@@@@@@@.                          
                @@@@@@@                           
                  @@@@                            
```

</small>