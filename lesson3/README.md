# Learning Mongo: Lesson 3: Building a help ticket database

In this directory you will find a help ticket web application. This includes a ticket server in `tickets.js` and a Vue front end for submitting tickets in the `public` folder. I recommend cloning
this repository so you can easily get this code.

Our database is really simple. A ticket contains a `name` and a `problem`.

We're going to add to this project so that it can store tickets in a Mongo database.

## Initialize the project

Let's initialize a new project. You do this with `npm init`.

```
cd lesson3
npm init
```

Now we need to install the libraries we need:

```
npm install express mongoose
```

## Try out the existing application

You can run the existing application with:

```
node tickets.js
```

## Configure mongoose

We first need to configure Mongoose. Modify `tickets.js` so it does this:

```
app.use(express.static('public'));

const mongoose = require('mongoose');

// connect to the database
mongoose.connect('mongodb://localhost:27017/test', {
  useNewUrlParser: true
});
```

## Create a model for tickets

Immediately after this, in `tickets.js`, create a model for a Ticket. We first
create a schema:

```
const ticketSchema = new mongoose.Schema({
  name: String,
  problem: String,
});
```

Next, we create a virtual parameter for the schema called `id`:

```
ticketSchema.virtual('id')
  .get(function() {
    return this._id.toHexString();
  });
```

We're doing this because by default every document in Mongo has a property called `_id`,
but our Express API and our Vue code uses `id` instead. This lets us easily convert
`_id` into `id`.

We also ensure that when documents are serialized into JSON objects that virtual
properties are included:

```
ticketSchema.set('toJSON', {
  virtuals: true
});
```

Finally, we create a model for tickets that uses this schema.

```
const Ticket = mongoose.model('Ticket', ticketSchema);
```

Since we are going to store tickets in the database, you can delete these lines:

```
let tickets = [];
let id = 0;
```

## Reading tickets

Now we need to modify the endpoint for reading tickets so it instead gets tickets
from the database:

```
app.get('/api/tickets', async (req, res) => {
  try {
    let tickets = await Ticket.find();
    res.send(tickets);
  } catch (error) {
    console.log(error);
    res.sendStatus(500);
  }
});
```

Notice that we use the async keyword when defining the function that is called
for this endpoint. This lets us call `await` for the Mongoose database query.
All Mongoose functions are Promises, so you can either use the `then/catch`
syntax or use `await`.

Note that when you use `await` you should wrap it in a `try/catch` block to
catch errors.

## Creating tickets

We also need to modify the endpoint for creating tickets so it adds tickets
to the database:

```
app.post('/api/tickets', async (req, res) => {
    const ticket = new Ticket({
    name: req.body.name,
    problem: req.body.problem
  });
  try {
    await ticket.save();
    res.send(ticket);
  } catch (error) {
    console.log(error);
    res.sendStatus(500);
  }
});
```

## Deleting tickets

Finally, we need to modify the endpoint for deleting tickets to it deletes
them from the database:

```
  try {
    await Ticket.deleteOne({
      _id: req.params.id
    });
    res.sendStatus(200);
  } catch (error) {
    console.log(error);
    res.sendStatus(500);
  }
```

Notice that we use `req.params.id` (the virtual parameter we sent when reading tickets)
from the database and map this to `_id`.

## Test it

The application should be done! It now saves data to the database instead of
to a local array. You can kill and restart the server and your data is permanently
stored.
