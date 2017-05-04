Node
Express
Handlebars
Knex
Postgres

# Create server
### Create folder
1. express --git --hbs
 - creates a .gitignore for you
 - sets view engine as Handlebars
2. Install dependencies
 - npm install
 - npm install --save knex pg
 - npm install
 - knex init
3. npm start
 - or __nodemon__ to start server

# Set up database
### Create DB
1. createdb todo_list
2. In the Knexfile.js
 - Set client to postgresql
 - Set connection for newly created, local database
```
development: {
  client: 'postgresql',
  connection: 'postgres://localhost/todo_list'
}
```
3. Wipe out everything below this -- for now.

### Create table
1. knex migrate:make 01_todo_list
2. Edit the newly created migration file to have:
  - Required (notNullable) fields of proper type
  - Any default values

  ```
    exports.up = function(knex, Promise) {
      return knex.schema.createTable("todo", (table) => {
        table.increments();
        table.text("title").notNullable();
        table.integer("priority").notNullable();
        table.text("description");
        table.boolean("done").defaultTo(false).notNullable();
        table.datetime("date").notNullable();
      });
    };

    exports.down = function(knex, Promise) {
      knex.schema.dropTable("todo");
    };

  ```
3. knex migrate:latest
  - confirm further by:
    1. psql todo_list
    2. __\dt__
    3. __\d todo__

### Seed table
1. knex seed:make 01_todo
2. Edit the newly created seed file:
  - Change the table_name accordingly
  - Create seed objects
 ```
   exports.seed = function(knex, Promise) {
     // Deletes ALL existing entries
     return knex('todo').del()
       .then(function () {
         // Inserts seed entries
         return knex('todo').insert([
           {
             title: "Buid a CRUD app",
             priority: 1,
             date: new Date()
           },
           {  title: "Do the dishes",
             priority: 3,
             date: new Date()
           },
           {  title: "Render a view",
             priority: 2,
             date: new Date()
           },
           {  title: "Walk the dog",
             priority: 5,
             date: new Date()
           }
         ]);
       });
   };
 ```
 3. knex seed:run
   - confirm further by:
     1. psql todo_list
     2. select * from todo;

# List all records with GET/todo
1. Duplicate index.js within Atom, and name todo.js
2. Edit routes in app.js
 - Create router
  ```
    //the variable todo is equal to the todo.js router
  ```
  var todo = require('./routes/todo');

 - Configure for use
  ```
    //when app.js sees the /todo URL, the express router is to then use the todo router defined above
  ```
  app.use('/todo', todo);

 - Configure todo.js
   1. Change title to "Todo"
   2. Check using browser:
   __localhost:3000/todo__

3. Create a DB folder
 - Create knex.js file to use knex to connect to todo_list database.

  ```
  const environment = process.env.NODE_ENV || "development";
  ```
  // when node runs, if the NODE_ENV variable is set (like in "production" Heroku environment) that environment will be used; otherwise, the "development", local environment.
4. Create production env in knexfile.js
  ```
  production: {
    client: 'postgresql',
    connection: process.env.DATABASE_URL + '?ssl=true'
  },
  ```
  - the addition of ?ssl allows for addition of secure migrations/seeds locally
5. Set the knex.js file to require the proper environment
  ```
  const config = require("../knexfile")[environment];
  ```
    - grabs an object in knexfile.js
    - specifies the environment
6. Create "config" for connecting to database
  ```
  module.exports = require("knex")(config);
  ```
    - "config" is equal to the whole "development" object in knexfile.js
    -knex.js file now exports a route
7. Add new route to todo.js file as active connection to database
  ```
  const knex = require("../db/knex");
  ```
8. Create a function to get the list of (all) todos
  - hey knex, goto the todo table, select all the rows,
  then give back todos, then I want to render all those on a page called "all"
  - let's create a page called "all", where I will have access to _property_ called "todos", with a _value_ in the form of an array of todos I just got back from the database.
  ```
  router.get('/', function(req, res, next) {
    knex("todo")
    .select()
    .then(todos => {
      res.render('all', { todos: todos});
    });
  });
  ```
9. Create new file all.hbs in Views, so we can actually see the list as we iterate over the todos.
  ```
    {{#each todos}}
      <h2>{{title}}</h2>
    {{/each}}
  ```
# Add Bootstrap
1. Add Bootstrap CDN to layout.hbs (above styles.css)
```
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
```
  - layout is the container for all the views that will be rendered by HBS.
2. Change all.hbs to use Bootstap list-group, so it looks better.
```
<ul class="list-group">
  {{#each todos}}
    <li class="list-group-item">{{title}}</li>
  {{/each}}
</ul>
```
  - _note that the {{each}} is placed before the <li>, rather than on the parent <ul> or <ol>_
  - changed the <li> text to contain {{title}}
3. Add a margin-top: 3em; to the body in styles.css (for readability)

# Create a form for "new" todos
### Create a route for "new" page
- We are generating, so we do not need anything from the database
- We also do not need to pass anything into the render
```
router.get('/new', function(req, res, next) {
  res.render('new');
});
```

### Create a "new" View
_to be rendered at /todo/new_
```
<form class="form" action="/todo" method="post">
  <div class="form-group">
    <label for="title">Title</label>
    <input type="text" class="form-control" id="title" placeholder="Enter a Todo">
  </div>
</form>
```

### Create a button on all.hbs page
- For new todo create/submit action
- Set class as btn-primary as a css color selection
- Points to "new" page on click
_as an <a> tag so it is clickable link_

```
<a type="button" class="btn btn-default btn-primary" href="/todo/new">Create New Todo</a>
```

### Create textarea in form (new.hbs) for "description"
```
<div class="form-group">
  <label for="description">Description</label>
  <textarea type="text" class="form-control" id="description" placeholder="Enter a Description">
  </textarea>
</div>
```
### Create dropdown in form (new.hbs) for "priority"
```
<div class="form-group">
  <label for="priority">Priority</label>
  <select class="form-control" id="priority">
    <option>1</option>
    <option>2</option>
    <option>3</option>
    <option>4</option>
    <option>5</option>
  </select>
</div>
```
### Create button in form (new.hbs)
_wound up putting in a <div> to get to render_
- For "submit"
  - Form action (POST) is addressed in next section
- Set class as btn-success as a css color selection
```
<button type="submit" class="btn btn-default btn-success">Create</button>
```

# Create record with POST/Todo
### Create route in todo.js
_remember that routes in todo.js are prepemded with /todo_
1. Create POST route
  - Validate data
    - Create validation function that returns if:
      1. Todo title is a string
      2. Title string is __not__ empty
      3. Remove the whitespace around it
      4. Priority __is__ a number

    ```
    function validTodo(todo) {
      return typeof todo.title == "string" &&  todo.title.trim() !== "" && typeof todo.priority == "number";
    }
    ```
  - router.POST (at this point)
    1. Send todo to server
    2. Validate it; make sure it has the properties that we want
    3. If so,
      - Create a todo object based on those properties from the body
      - Insert that todo into the database where equal to the id
        1. grab the first ID
        2. Redirect to a View/page to view the thing just created
    4. If not,
      - Return status: 500
      - Render to error.hbs
        1. Create object to include what page expects
        message:
        2. __Note__ that the redirect path uses backticks (not single quotes)

  ```
  router.post('/', function(req, res, next) {
    console.log(req.body);

    //the incidental logging here shows the empty object, and indicates a need for input "name" values so something can be passed to server

    if(validTodo(req.body)) {

      //create todo

      const todo = {
        title: req.body.title,
        description: req.body.description,
        priority: req.body.priority,

        //setting this way addresses constraint by pumping current date into form - see CRUD readme for more detail

        date: new Date()
      };

      //this portion not included in function, but rather pulled out and altered for the POST (insert) and PUT (update) routes accordingly
        //insert, then immediately redirect to the {id}
        //or update, then immediately redirect to the proper page

      knex('todo')
      .insert(todo, 'id')
      .then(ids => {
        const id = ids[0];

        //check into why not affected by todo prepend

        res.redirect(`/todo/${id}`);
      });
    } else {

      // respond with an error

      res.status(500);
      res.render('error', {
        message: 'Invalid Todo'
      });
    }
  });
  ```

# Give the form inputs "name" values
- Running the above, "as-is" returns an empty object
- Edit validation function in todo.js accordingly:
 ```
  function validTodo(todo) {
    return typeof todo.title == "string" && todo.title.trim() != "" && typeof todo.priority != "undefined" && !isNaN(Number(todo.priority));
  }
 ```
- Adding name values solves one problem, __but__
  - Did require a reboot to see some values
  - Log shows requirement for date field to be not-null
  - Added "date" to address new log error
    - Added date to todo object to accommodate entry of values to db; discovered above
    ```
    date: new Date();
    ```
    - Had not been able to set a default value to prevent "notNullable" error earlier within knex migrate
- Though log still throws a 404 error, there should be a new record showing on the index.
  - Need GET / todo route.

# Show one record with GET/todo/:id
See todo.js for '/:id' route function
  - May seem counter-intuitive, but we are working backward, because we want to work with a specific id to get at a single todo...__so__
    - If (validate that) the type of id is not undefined
    - Have knex look it up in the todo table
    - Select everything in the row where the id matches the one we want
      - only care about the first one
      - render it on the "single"  View
    - __else__ send back an error
  - Big Picture - we want each of the entries on the "index" page to link to their corresponding "single" View entry, the router.get would be something like:
  ```
  router.get('/:id', function(req, res, next) {
    const id = req.params.id;
    if(typeof id != "undefined"){
      knex("todo")
      .select()
      .where("id", id)
      .first()
      .then(todo => {
        console.log({todo: todo});
        res.render('single', {todo: todo});
      });
    } else {
      res.status(500);
      res.render("error", {
        message: "Invalid Todo"
      });
    }
  });
  ```

  - Change the <li> elements on all.hbs to <a> so they are linkable
  - Add href="/todo/{{id}}" to represent the current todo's id
    ```
    <a href="/todo/{{id}}" class="list-group-item">{{title}}</a>
    ```

### Create "single" View
1. Create new single.hbs file
2. Selected panel as single.hbs output format.
  - Included HBS values for title/description
  - Added a badge for priority
  - Put date into footer
  ```
  <div class="panel panel-default">
    <div class="panel-heading">
      <h3 class="panel-title">{{title}}   <span class="badge">{{priority}}</span></h3>
    </div>
    <div class="panel-body">
      {{description}}
    </div>
    <div class="panel-footer">{{date}}</div>
  </div>
  ```
  - Frankenstein'd together a View but, wound up empty
    - Added console.log to todo.js "/:id" route to see what todo actually returned
    - Around 47 minutes, discussion of passing __todo__ to the route above rather than {todo: todo} because the object already contains all the necessary information so as to no have to add todo to each of the (4) handlebar items in the "single" View.  Bit/myHead...but current version looks like:
  ```
  router.get('/:id', function(req, res, next) {
    const id = req.params.id;
    if (typeof id != 'undefined') {
      knex('todo')
        .select()
        .where ('id', id)
        .first()
        .then(todos => {
          res.render('single', todo);
        });
    } else {
      res.status(500);
      res.render('error', {message: 'Invalid Id'});
    }
  });
  ```

# Show an edit form with GET / todo/:id/edit
1. Create edit route
_remember that routes on todo.js prepended by /todo_
  ```
  router.get('/:id/edit', function(req, res, next) {
    res.render('edit');
  }
  ```
2.  Create clickable button on single.hbs
 - Id edited to interact with Handlebars
 - For directing to "edit" View, which will be created shortly
  ```
  <a href="/todo/{{id}}/edit" type="button" class="btn btn-default btn-warning">Edit todo</a>
  ```
  3. Create edit.hbs (duplicate new.hbs)
    - Give each of the inputs a value=""
      - Same as their name
      - in Handlbars format (e.g. value="{{title}}"  )
    - Change button
      - Make text Update, rather than Create
      - Change class to btn-danger (red), rather than btn-success (green)
4. We want to see data for each record, so need a route to get info from DB
  - Realize that the router.get('/:id') route contains the same logic, so turn into function
  ```
  function validateTodoInsertUpdateRedirect(req, res, callback) {
    if(validTodo(req.body)) {
      //create todo
      const todo = {
        title: req.body.title,
        description: req.body.description,
        priority: req.body.priority,

      };

      callback(todo);
      } else {
        // respond with an error
        res.status(500);
        res.render('error', {
          message: 'Invalid Todo'
        });
      }
    }
  ```
  - Adjusted router.get('/:id') for both 'single' and 'edit' routes to use respondAndRenderTodo function
  ```
  router.get('/:id', (req, res) => {
    const id = req.params.id;
    respondAndRenderTodo(id, res, 'single');
  });

  router.get('/:id/edit', (req, res) => {
    //get the todo with the id in th URL
    const id = req.params.id;
    respondAndRenderTodo(id, res, 'edit');
  });
  ```
  - Title should be updating, but priority not selecting accordingly
5. Steps taken to deal with priority
  - Tried setting values on each
  - Tried changing the input value= to selected=, instead (per SO recomm).
  - Suggestion to employ 'selected' attribute
  - Rambling thoughts:
    - How to address with Handlebars?
    - Possibly need to use a conditional
    - Could use JS, but doing server-side rendering, so want to avoid using client-side JS
  - __Solution:__ found SO post with HBS helper
    - Required HBS in app.js
    - Included helper in app.js
    ```
    var hbs = require('hbs');

    hbs.registerHelper('select', function(selected, options) {
        return options.fn(this).replace(
            new RegExp(' value=\"' + selected + '\"'),
            '$& selected="selected"');
    });
    ```
    - Changed elements edit.hbs
      1. Handlbars select complement to helper (above)
      2. Also, __need__ value={{priority}} to capture the id
    ```
    <div class="form-group">
      <label for="priority">Priority</label>
      <select class="form-control" value="{{priority}}" name="priority" id="priority">
        {{#select priority}}
        <option value="1">1</option>
        <option value="2">2</option>
        <option value="3">3</option>
        <option value="4">4</option>
        <option value="5">5</option>
        {{/select}}
      </select>
    ```

# Update record with PUT /todo/:id
### Work on PUT route to /:id
1. Realize that the router.post('/:id') route contains the same logic, so turn into function, instead, as well
  - Create function - refactor
    1. Validate todo
    2. Needs access to request, response, and redirect (URL to)
      - The redirect replaced by callback function that takes in the (todo)
      ```
      callback(todo)
      ```
    3. Date pulled out of function and dealt with as part of PUT request (and affected POST).
    4. Use req.params.id to identify and specify id for redirect
2. Abstracted (tortured) function:
```
function validateTodoInsertUpdateRedirect(req, res, callback) {
  if(validTodo(req.body)) {
    const todo = {
      title: req.body.title,
      description: req.body.description,
      priority: req.body.priority,
    };

    callback(todo);

  } else {
    res.status(500);
    res.render('error', {
      message: 'Invalid Todo'
    });
  }
}
```

### Resulting router.put
__NOTE__ the redirect uses backticks (not single quotes)
```
router.put('/', function(req, res, next) {
  validateTodoInsertUpdateRedirect(req, res, (todo) => {

    //did not want to set date each time updated, so pulled out of function
    //decided to deal with later
    //todo.date = new Date();

    knex('todo')
      .where("id", req.params.id)
      .update(todo, 'id')

      //ids not needed, can just use {req.params.id}

      .then(() => {

        //const id = ids[0];     not needed; does not give back id, so

        res.redirect(`/todo/${req.params.id}`);
    });
  });
});
```

### Resulting router.post due to function extraction
```
router.post('/', function(req, res, next) {
  validateTodoInsertUpdateRedirect(req, res, (todo) => {
    todo.date = new Date();
    knex('todo')
      .insert(todo, 'id')
      .then(ids => {
        const id = ids[0];
        res.redirect(`/todo/${id}`);
    });
  });
});
```

### Change to edit.hbs required to accommodate the PUT
- Change action to reflect new path
```
action="/todo/{{id}}"
```
- However, changing form method to PUT did not work as expected, because HTML page can only POST or GET
  1. Method Override - Middleware required to allow for form PUT, UPDATE, DELETE
    - Need to install package
    ```
    npm install --save method-override
    ```
    - Require in app.js
    ```
    var methodOverride = require('method-override')
    ```
    - Add to app.use section, but make sure also above the /todo route so it is loaded by the time we need it
    ```
    app.use(methodOverride('_method'))
    ```
  2. Usage requires "?", so form tag now looks like:
  ```
  <form class="form" action="/todo/{{id}}?_method=PUT"  method="post">
  ```

# Delete a record with DELETE /todo/:id
1. Create a delete route
  - When I see a delete request on /:id
  - Validate / make sure valid ID
    - Else respond with an error
  - Find the todo with that ID (aka matches)
  - Delete
  - Not going to get anything back
  - Then redirect the "todo" page
2. Edit "single" page
  - Add Delete button
    - Copied edit button as template
    - Changed to btn-danger for color change
    - Remove href
  - Add hidden form
    - Edit form action to allow for DELETE method (override)
    ```
    <form class="" action="/todo/{{id}}?_method=DELETE" method="post">
    ```
    - Change from < a >-tag to actual <button>
    - Move button inside form
    - Convert button to type="submit"
  - Moved the edit < a >-tag button inside the form
    - Form is block level, so putting buttons together allows for them to sit on same line - looks better
    - The edit only redirects to the edit page, only the delete button performs "submit" on the form - no conflict.

### Resulting router.delete
```
router.delete('/:id', (req, res) => {
  const id = req.params.id;
  if(typeof id != "undefined"){
    knex("todo")
    .where("id", id)
    .del()
    .then(() => {
      res.redirect('/todo');
    });
  } else {
    res.status(500);
    res.render("error", {
      message: "Invalid Todo"
    });
  }
});
```


# end game walkthrough
1. Creating new todo - redirects to "single" view page
  - Discover error: Unhandled rejection error: invalid input syntax for integer: "new"

2. Editing todo - redirects to "single" view page
3. After deleting todo - redirects to 'all'
