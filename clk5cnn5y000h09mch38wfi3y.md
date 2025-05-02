---
title: "Benefits  of Lazy Loading image and how to do it in react easily ?"
datePublished: Sun Jul 16 2023 11:24:42 GMT+0000 (Coordinated Universal Time)
cuid: clk5cnn5y000h09mch38wfi3y
slug: benefits-of-lazy-loading-image-and-how-to-do-it-in-react-easily
tags: optimization, tips, reactjs, lazyload

---

## Why Lazy Load

* Improved performance: It allows you to defer the loading of images until you need it resulting in saving time and fast page rendering. It only loads the images when they are in or near the viewport
    
* Bandwidth Optimization: By lazy loading images, you can reduce the amount of data transferred to the user's device, especially for long web pages with multiple images.
    
* Faster perceived load time: Lazy loading images can enhance the perceived performance of your website. Instead of waiting for all images to load before displaying any content, you can show the main content first and progressively load images as the user scrolls, providing a better user experience.
    
* Better user engagement: Faster page load times and a smoother browsing experience can lead to improved user engagement and satisfaction. Users are more likely to stay on your website and explore further if they are not waiting for images to load.
    

## How to implement it?

To implement lazy loading of images in React, you can use various libraries or techniques. One popular library is react-lazy-load-image-component, which simplifies the process. Here's a step-by-step guide on how to use it:

1. Install the react-lazy-load-image-component package via npm or yarn:
    
    ```plaintext
    npm install react-lazy-load-image-component
    
    // or
    
    yarn add react-lazy-load-image-component
    ```
    
2. Importing the library into your component
    
    ```javascript
    import { LazyLoadImage } from 'react-lazy-load-image-component';
    import 'react-lazy-load-image-component/src/effects/blur.css'; // If you want to use the blur effect
    ```
    
3. Replace your regular \`img\` tags with the LazyLoadImage component. Set the 'src' attribute to the actual image and add an attribute placeholderSrc providing the placeholder image or a low-resolution version of the actual image:
    
    ```javascript
    <LazyLoadImage
            effect="blur" // Optional: adds a blur effect while loading
            src={highResolutionImage} // Actual image source
            alt="Description of the image"
            placeholderSrc={placeholderImage} // Placeholder image or loading indicator
          />
    ```
    

You Have SuccessFully Implemented LazyLoading In your component .