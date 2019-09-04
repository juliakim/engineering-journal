# FEC Engineering Journal and Notes

The good, the bad and the ugly. Journaling the joys and frustrations of creating a clone of Amazon employing Service Oriented Architecture.

## Module Specifications

### Which module did you choose? Why?

I am responsible for the photo gallery. I am excited to gain exposure to media-centric widgets and explore the intricacies of optimized image rendering.

### What will you have to do to recreate this module?

*What is it's main functionality or purpose?*

The photo gallery module provides the end user with a collection of images representing the product of interest. The user has the ability to select from these images.

*What are important design/styles to create for the user experience?*

When an image is selected by the user, the main photo on the page must be changed accordingly. Additional important considerations for the user experience include the zoom in on hover and expanded view on click functionalities.

*Base Goals*
* Add a high resolution product image with alternative images along left of page
* Change to desired alternative image on hover or click
  * Add orange outline around border of selected image
* Roll over to zoom functionality
  * New on-page close-up of item on right
  * Relevant part of image in dotted blue square within image boundaries
* Text below image alternating between 'Roll over image to zoom in' and 'Click image to open expanded view'

*Stretch Goals*
* Include videos
* Expanded view pop-up
  * Title/color of item
  * Images
  * Related videos

## Optimizing Performance

### What can be optimized? What are the trade-offs?

### Performance Notes

What did you try?

What is the difference in performance?

Does the page load faster or create a better user experience?

## Proxy Server

### How do you get all your modules to display together?

## Other Notes

### Error Messages / Resources / New Techniques

Deployment - coming soon

## Daily Reflections

### Day 1

We met as a group to divide modules and design the layout of the page. I did some initial research on Git workflow, scanned the Airbnb style guide and got oriented with Trello. I also sketched a rough outline for my component. I learned about feature branch workflow which involves creating new branches when starting work on new features.

Overall, it felt like a slow start. Because I have the tendency to research until mastery, I need to resist the urge to browse every resource from a search as this is a time sink.

### Day 2

To start off the day, I browsed through MongoDB and Mongoose documentation and related blogs. Installing MongoDB proved much more difficult and time-consuming than initially anticipated. As with all things, I believe I overcomplicated this by piecing together instructions from various sites instead of simply consulting documentation. I still have an aversion to documentation, so I definitely plan to work on this moving forward.

In the afternoon, I registered for S3 and uploaded a few images with which I created a sample database and collection using Compass. I set up React, Webpack, Babel, Express and MongoDB to begin passing data from the client to server to database and back.

### Day 3

React is now receiving dummy data from MongoDB, specifically links to images on S3, and rendering it to the page. To my existing MediaContainer component, I added the following components: AdditionalMedia, Caption, Image, MainImage and Video. Using my wireframe as a guide, I have nested some components within others, successfully passing state down.

The better part of my day was spent uploading images to S3 and creating a guide for my team so that they could do the same. In addition, I read through the first part of a Medium article on deployment utilizing EC2 and PM2. I am currently running an instance on EC2.

### Day 4

I started my day with light reading on flexbox then jumped into styling my components. Though fun, CSS is challenging due to the vast number of available properties. By the end of the day, my module somewhat resembles Amazon's image gallery with a few issues needing to be fine-tuned.

### Day 5

I continued to work with CSS to style my component. I struggled with setting dimensions and maximum dimensions. The biggest challenge - that is yet to be solved - was making the images responsive to the size of the window. By the time my component looked nearly identical to Amazon's (by brute force), I realized my code had gotten dense and convoluted. As I added code to tweak a feature, another component broke or lost its position or appearance. I am considering starting afresh, individually adding rules from my current stylesheet to filter through unnecessary lines.

### Day 6

I proceeded with creating a new stylesheet, referencing individual rules from my former file. I found much success in doing so as this allowed me to eliminate extraneous property declarations. I set width and height of images as percentages to address yesterday's issue with image resizing, relying heavily on Chrome DevTools - notably the device toolbar - to ensure my images were responsive.

Image scaling is a feature I hope to revisit before the conclusion of this project. Using the CSS object-fit property, I was able to resize the main image to fit inside its container while maintaining its original aspect ratio. This, however, did not work as desired for landscape images which were enlarged to meet the 'height: 100%' property. I believe I can correct this by adding an orientation-dependent class to the image in React and specifically tailoring width and height properties to each.

On the backend, I seeded my database using the `Model.insertMany()` function. I deliberated file structure for quite a while before separating server requests from database queries. I remain confused when it comes to MVC architecture but plan to reorganize my files if time permits. I also came across an issue with the `Model.find()` function, thinking it returned an object. Upon reading documentation, I learned that this function returns a mongoose Query object which can be executed by either (a) passing in a callback function or (b) calling the `.then()` function which allows the object to be used as a promise.

### Day 7

The majority of my day was devoted to researching how to zoom on hover. Because there is no native event handler for hover in React, I combined `onMouseEnter`, `onMouseMove` and `onMouseLeave` to achieve a similar effect. I tied these mouse events to a function that tracks coordinates of the mouse when within the frame of the MainImage component.

Next, I faced the task of finding coordinates relative to the image which, I imagine, requires the dimensions of the image. This was complicated by the `object-fit: contain` property used to resize the image. I could not discover a way to access the dimensions of images (see below, yellow outline) that were scaled down to fit in a container (red outline).

![Differing Dimensions](differing_dimensions.png "Image vs. Container")

At this point, I am debating using a magnifier library - will implementing from scratch be a rewarding learning experience or will my time be better spent coding a different feature? On a different note, I have migrated my database from my local instance of MongoDB to Atlas to prepare for deployment. I also deployed my client and proxy server, both of which are running on separate EC2 instances. I will need to do further research on how to authorize requests to Atlas from AWS.

### Day 8

I divided my time between creating an expanded view modal and embedding videos for the few products with links. The modal was relatively simple to set up with the exception being the click-outside-to-close function and dimmed background. I created a new component, ExpandedView, to serve as the popover that appears when the main image is clicked. To this, I had to add a temporary event listener and ref to prevent closing the modal from clicks within it. I emulated the dimmed background effect by adding a shadow to the modal - 'hacky' but will do for now.

After implementing the modal, I noticed that the orange selected item border disappeared upon my mouse leaving the thumbnail. I should have considered this when debating between using Javascript or CSS for this effect. I will eventually refactor this with React.

### Day 9

After experimenting with CSS, I styled the expanded view overlay to resemble Amazon's, primarily using flexbox to position components. I conditionally tagged components with IDs to render different pages depending on video availability. At this time, the overlay does not scale with window size.

### Day 10

To keep consistent with Amazon, I reexamined the outline on hover effect that did not persist when the overlay was opened. Instead of using the `:hover` selector through CSS, I added a conditional ID and another property, hoveredThumbnail, to the state of my top level React component to track the selected thumbnail. This solved the issue with losing the border.

Previously, I noticed that the overlay did not prevent state changes in other components.

![Changes in background](https://media.giphy.com/media/SqZ9TGwrwGjQLF2BJv/giphy.gif)

I wrapped the ExpandedView component in a div to prevent mouse events from falling through to the elements behind it. This div is positioned below the popover but above all other components. I also removed the modal's shadow and instead added a light gray background to this div to preserve the shaded effect.

![Prevent state changes in background](https://media.giphy.com/media/H6bzsFknsoLkadATt6/giphy.gif)

### Day 11

I completed most of the CSS styling for the modal, positioning most components in fixed places.

### Day 12

I mentioned previously that the modal as a whole does not perfectly scale - this is precisely because it was being appended to the same div as the MediaContainer component whose width was set to 50%. I needed to find a way for the modal to receive props from it's parent component, MediaContainer, while maintaining a width of 100%. In researching this issue, I discovered portals, a React 16 feature that allows the user to render a child component to a DOM node outside the current DOM hierarchy, as the perfect remedy. This would allow me to render the modal to a different div without a preset width all the while continuing to propagate to the parent component.

I also made changes to CSS in order to make the contents of the modal responsive. Most of this was achieved using viewport units and `calc()` with media queries to set a maximum font size.

Lastly, I reconfigured my EC2 instance to serve my HTML file using nginx - to route port 80 traffic to my desired port - and PM2 - to keep the node.js process running.