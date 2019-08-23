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