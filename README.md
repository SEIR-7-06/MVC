# MVC

## Lesson Objectives

1. Define MVC and explain why it matters
1. Move our data into a separate file
1. Move our presentation code into EJS files


## Define MVC and explain why it matters

One of the core tenants of good programming is to compartmentalize your code.

Our `server.js` file is getting messy since it contains:

- data
- app instantiation (listening)
- routes

### Defining MVC

One way to keep an app from getting messy is to separate it out into three sections:
- **Models**: communicates with our database, and determines what the data should look like
- **Views**: how the data is displayed to the user (HTML)
- **Controllers**: responsible for responding to client requests, which often involves rendering the models (i.e., data) and the views (i.e., HTML)

### Specializations

This allows various developers to divide up a large code base, which:
- minimizes the likelihood of developers overwriting each others' code.
- allows developers to specialize.
    - One can focus on dealing with data
    - One can focus on html
    - One can focus on connecting the two

### A metaphor

Think of MVC as a restaurant.
- Models are the cook, which prepares food/data
- Views are the platter on which the food/data is presented for consumption
- Controllers are the waiter, which:
    - receives the request from the customer/browser
    - asks the cook to prepare the food/data (and has no idea how food/data is prepared)
    - takes the food/data and combines it with the platter that are pre-built to hold food/data
    - fulfills the customer's/browser's request by delivering the platter full of food/data (and also has no idea how the food/data is consumed)

## Setup

1. `cd ~/sei/express-fruits`
2. Open `express-fruits` in your editor.
3. `nodemon server.js`

## Move our data into a separate file

1. `mkdir models`
2. `touch models/fruit_model.js`
3. Move the `fruits` array from `server.js` to `models/fruit_model.js`:

```javascript
const fruits = [
    {
        name:'apple',
        color: 'red',
        readyToEat: true
    },
    {
        name:'pear',
        color: 'green',
        readyToEat: false
    },
    {
        name:'banana',
        color: 'yellow',
        readyToEat: true
    }
];    
```

4. We still need access to the fruits data in `server.js`, so we'll need to import it. This is what the `require()` function does in NodeJS. (Note: If we're importing another file, we use a relative path, typically starting with `./`. If we're importing an npm package, the `require()` function knows to look in the `node_modules` directory for a subdirectory of that exact name.)

In `server.js`:

```javascript
const fruits = require('./models/fruit_model.js');
```

5. We still haven't imported anything yet because we haven't told the `models/fruit_model.js` what to export. Like any JS file, we can have multiple variables/functions that can be exported.

So, how does JavaScript know which variable(s) in `models/fruit_model.js` to assign to the fruits `const` in `server.js` (i.e., what exactly is the `require()` statement importing)?

We must tell JavaScript which variable(s) we want to export so that other files, such as `server.js`, can import them.

In NodeJS, every file is also called a "module." And, each module/file has a built-in object called `exports` that the `require()` method reads. We need to add `fruits` to this object.

In `models/fruit_model.js`:

```js
// at the bottom
module.exports = fruits;
```

## Create the Show Fruit Page

Now we want to create our View code (HTML) in a separate file just like we did with the data.

1. Install EJS (Embedded JavaScript), our second NPM package: `npm install ejs`
    - This is a templating library that allows us to use JavaScript to mix data into our HTML. This means that the HTML will change based on the data!
2. First, we need to create a directory called `views`: `mkdir views`.
3. `touch views/show.ejs`
    - This will be the html for our show route.
4. Put some html into `show.ejs`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <h1>Fruits show page</h1>
</body>
</html>
```

5. Now, instead of `res.send('some text')`, we can call `res.render('show.ejs')`.
    - Express will know to look inside the `views` directory.
    - It will send the html in the `show.ejs` file as a response to the browser's request.

    ```js
    app.get('/fruits/:fruitIndex', function(req, res){
        res.render('show.ejs');
    });        
    ```

Now let's mix our data into our HTML.

6. In `server.js`, let's modify our show route to gather the data and pass it to the view:

    ```js
    app.get('/fruits/:fruitIndex', function(req, res){
        // the first param of render() is the .ejs file that we want to inject data into
        // the second param is the data that we want to inject into the .ejs file (it must be an object)
        res.render('show.ejs', {
             //there will be a variable available inside the show.ejs file called oneFruit, and its value is fruits[req.params.fruitIndex]
            oneFruit: fruits[req.params.fruitIndex]
        });
    });    
    ```

7. Access the data that `res.render()` made available to the view.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>Fruits show page</h1>
    <p>The <%= oneFruit.name %> is <%= oneFruit.color %>.</p>
    <% if(oneFruit.readyToEat === true){ %>
        <p>It is ready to eat</p>
    <% } else { %>
        <p>It is not ready to eat</p>
    <% } %>
</body>
</html>
```

Up until now, if we wanted to create JavaScript in an HTML file, we'd have to do so between `<script>` tags. Now, we're just writing JavaScript in the middle of the HTML, and this is thanks to EJS and its templating tags.

We know that `oneFruit` is an object that refers to one of the fruit objects (e.g., `{ name:'apple', color: 'red', readyToEat: true }`) from the `fruits` array. If `oneFruit` represents one of those objects (depending on which index is passed as a URL parameter), then we know that we can reference its properties as, for example, `oneFruit.name`.

We want the value of `oneFruit.name` to show up on the page. We don't, however, want the value of `oneFruit.readyToEat` to show up on the page. Instead, because it's a boolean, we want to use it to determine which statement ("It is ready to eat" or "It is not ready to eat") becomes part of the final, rendered HTML that is sent back to the client/browser.

Note that there are two types of new tags:
- `<% %>`: run some JavaScript
- `<%= %>`: run some JavaScript and insert the result of the JavaScript into the HTML

## Create the Index Fruit Page

So far, we were able to create a "Fruit Show Page", a page that will show the information about a single fruit. We created a `show.ejs` template, rendered that template in the fruit show route, and passed it the data for a single fruit.

Our next goal is to create a page that will show the information for all the fruits in our database. This will be our "Fruit Index Page".

1. Start by creating a file called `index.ejs` in the `views` directory.

2. We'll start off with some boilerplate HTML in the `index.ejs` file with an h1 tag.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <h1>Fruit Index Page</h1>
</body>
</html>
```

3. We can then render out the `index.ejs` template in our show route.
```js
app.get('/fruits', (request, response) => {
  response.render('index.ejs');
});
```

At this point we should be able to visit `localhost:4000/fruits` in the browser and visit our "Fruits Index Page". Try it out!

4. The next step is the pass the fruits data to the `index.ejs` template.

```js
app.get('/fruits', (request, response) => {
  // response.send(fruits);
  response.render('index.ejs', {
    allFruits: fruits
  });
})
```

5. At this point we are able to render our `index.ejs` template and we are passing it the fruits data. That fruit data is accessible in the `index.ejs` template under the variable `allFruits`. `allFruits` is the key in the key/value pair being passed when we are rendering the `index.ejs` template.

We are now ready to display that data on the page. For each fruit, let's display the fruit name and the fruit color.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <h1>Fruit Index Page</h1>

  <ul>
    <% for (let i = 0; i < allFruits.length; i++) { %>
      <li>
        <p>The <%= allFruits[i].name %> is <%= allFruits[i].color %></p>
      </li>
    <% } %>
  </ul>
</body>
</html>
```

We are simply inserting a JavaScript for loop into our EJS template. For each fruit in the `allFruits` array, we printing out the fruit name and the fruit color.
