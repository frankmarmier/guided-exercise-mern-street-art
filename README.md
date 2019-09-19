# Guided Exercise | MERN Street Art

## Introduction

In this project, you will build a complete fullstack application where users go create/read/update/delete street arts.

A demo is available here: https://mern-street-arts.herokuapp.com (the map is not available)

This project will rely on the following technologies: 
- Front-end:
  - React
  - Bootstrap
  - Mapbox
  - Cloudinary
- Back-end:
  - Express
  - MongoDB and Mongoose


## Overview

### Wireframes on Figma
- [Wireframes on Figma](https://www.figma.com/file/ThXTvVPZaJz53bjMdURF3t/MERN-Street-Art?node-id=0%3A1)
- [Preview Wireframes on Figma](https://www.figma.com/proto/ThXTvVPZaJz53bjMdURF3t/MERN-Street-Art?node-id=15%3A28&scaling=scale-down)

### Wireframes on a board

To create the wireframes, we have followed some rules (feel free to adapt them for your own projects)
- Mobile first
- 1 screen = 1 wireframe
- Red = pages
- Blue = extra part when connected

![](./assets/wireframes.png)

### Models

For this project, you will use 3 models:
- `User`: To save basic information about the user (email and password)
- `StreetArt`: To save a picture and the location of a street art
- `Visit`: To save the visit of a specific user with a specific street art

#### `User`
Field     | Type
----------|------------------------------
_id       | (Automaticaly generated)
email     | String
password  | String
createdAt | Date (Automaticaly generated)
updatedAt | Date (Automaticaly generated)

#### `StreetArt`
Field          | Type
---------------|------------------------------
_id            | (Automaticaly generated)
pictureUrl     | String
location       | (GeoJSON)
-> type        | `"Point"`
-> coordinates | [Number]
createdAt      | Date (Automaticaly generated)
updatedAt      | Date (Automaticaly generated)

#### `Visit`
Field      | Type
-----------|------------------------------------------
_id        | (Automaticaly generated)
_user      | Schema.Types.ObjectId
_streetArt | Schema.Types.ObjectId
createdAt  | Date (Automaticaly generated at creation)


### Backend routes

Route                               | Type of protection | Description
------------------------------------|--------------------|-------------------------------------
`POST /api/signup`                  | Ø                  | Sign up the user
`POST /api/login`                   | Ø                  | Log in the user
`POST /api/logout`                  | Must be connected  | Log out the user
`GET /api/street-arts`              | Ø                  | Get all street arts
`GET /api/street-arts/:streetArtId` | Ø                  | Get the detail of one street art
`POST /api/street-arts`             | Ø                  | Add a street art
`GET /api/my-visits`                | Must be connected  | Get the visits of the connected user
`POST /api/visits`                  | Must be connected  | Add a visit
`DELETE /api/visits/:visitId`       | Must be the owner  | Delete a visit


## Iterations

- Initialise with the [MERN boilerplate](https://github.com/mc100s/mern-hooks-boilerplate)
- Create the models
- Create a seed file
- Create the backend routes
- Include Bootstrap in the project and customise it
- Create a page to see all street arts in a list
- Create a page to see all street arts in a map
- Create a page to add a new street art
- Deploy it

### Iteration 1 | Initialise with the MERN boilerplate

First, create a GitHub repository. We will assume the link is `<https://github.com/user/my-project.git>`

Open your terminal, go in a folder where you want to create the project and execute the following commands.
```sh
# Clone the project with only the last commit and save it in the folder <my-project>
$ git clone --depth=1 https://github.com/mc100s/mern-hooks-boilerplate.git mern-street-art

$ cd mern-street-art
$ rm -rf .git
$ git init

# Set your GitHub repository as the "origin" remote repository (remove < and >)
$ git remote add origin <https://github.com/user/my-project.git>
```

Create a file `server/.env`, with for example the following values;

```
PORT=5000
SESSION_SECRET=anyValue
MONGODB_URI=mongodb://localhost/mern-street-art
```

**To install all the packages**
```sh
# Install server and client packages + build the React applicatin
$ npm install

# OR you can install manually the server and client packages
$ (cd server && npm install)
$ (cd client && npm install)
```

**To install a package for the server**


```sh
$ cd server
$ npm install axios
```

**To install a package for the client**

```sh
$ cd client
$ npm install axios
```

**To run the server and the client**

```sh
# Open a first terminal
$ npm run dev:server
# Run the server on http://localhost:5000/

# Open a second terminal
$ npm run dev:client
# Run the client on http://localhost:3000/
```

So now you can go to

- http://localhost:5000/api/: A simple API call
- http://localhost:5000/: The website based on client/build (that you can update with `$ (cd client && npm run build)`)
- http://localhost:3000/: The last version of your React application that is calling your API with the base url "http://localhost:5000/api/"

### Iteration 2 | Backend |  Create the models and seed the database

For this part, we give you the code!

**Update the file `server/models/User.js`**
```js
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const schema = new Schema({
  email: String,
  password: String
}, {
    timestamps: true
  });

module.exports = mongoose.model('User', schema);
```

**Create a file `server/models/StreetArt.js`**
```js
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const schema = new Schema({
  pictureUrl: String,
  location: {
    type: { type: String, enum: ["Point"], default: "Point" },
    coordinates: [Number]
  },
}, {
    timestamps: true
  });

module.exports = mongoose.model('StreetArt', schema);
```

**Create a file `server/models/Visit.js`**
```js
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const schema = new Schema({
  _user: {
    type: Schema.Types.ObjectId,
    ref: 'User'
  },
  _streetArt: {
    type: Schema.Types.ObjectId,
    ref: 'StreetArt'
  },
}, {
    timestamps: {
      createdAt: 'createdAt'
    }
  });

module.exports = mongoose.model('Visit', schema);
```

**Update the file `server/bin/seeds.js`**
```js
const path = require('path')
require('dotenv').config({ path: path.join(__dirname, '../.env') })

// Seeds file that remove all users and create 2 new users

// To execute this seed, run from the root of the project
// $ node bin/seeds.js

const mongoose = require("mongoose");
const bcrypt = require("bcrypt");
const User = require("../models/User");
const StreetArt = require("../models/StreetArt");
const Visit = require("../models/Visit");

const bcryptSalt = 10;

require('../configs/database')

let userDocs = [
  new User({
    email: "alice@gmail.com",
    password: bcrypt.hashSync("alice", bcrypt.genSaltSync(bcryptSalt)),
  }),
  new User({
    email: "bob@gmail.com",
    password: bcrypt.hashSync("bob", bcrypt.genSaltSync(bcryptSalt)),
  })
]

let streetArtDocs = [
  new StreetArt({
    pictureUrl: "https://lh5.googleusercontent.com/p/AF1QipNqlBgeyUgKqGUH_oYogtxRQ0KPTtLAgiCXEUon",
    location: {
      type: "Point",
      coordinates: [ -9.209744,38.696060]
    },
  }),
  new StreetArt({
    pictureUrl: "https://lh5.googleusercontent.com/p/AF1QipO_kynLt94FYjYKmstOul5mZ-fnXyb6O_2Kr7SL",
    location: {
      type: "Point",
      coordinates: [-9.136864,38.720205]
    },
  }),
  new StreetArt({
    pictureUrl: "https://lh5.googleusercontent.com/p/AF1QipONkHmWhUjFjelUXxlekBg1Aq0ccW20yXxBRxxQ",
    location: {
      type: "Point",
      coordinates: [13.451661,52.507734]
    },
  })
]

let visitDocs = [
  new Visit({
    _user: userDocs[0]._id,
    _streetArt: streetArtDocs[0]._id,
  }),
  new Visit({
    _user: userDocs[0]._id,
    _streetArt: streetArtDocs[1]._id,
  }),
  new Visit({
    _user: userDocs[1]._id,
    _streetArt: streetArtDocs[0]._id,
  })
]


Promise.all([
  User.deleteMany(),
  StreetArt.deleteMany(),
  Visit.deleteMany(),
])
  .then(() => {
    console.log('All users, street arts and visits have been deleted')

    return Promise.all([
      User.create(userDocs),
      StreetArt.create(streetArtDocs),
      Visit.create(visitDocs),
    ])
  })
  .then(() => {
    console.log(`${userDocs.length} users created`)
    console.log(`${streetArtDocs.length} street arts created`)
    console.log(`${visitDocs.length} visits created`)
    mongoose.disconnect()
  })
  .catch(err => {
    mongoose.disconnect()
    throw err
  })
```

**Execute the seed**
```sh
$ node server/bin/seeds.js
```

If the command is not working, you probably have to run the following command in a terminal (without killing it)

```sh
$ mongod
```

**Check in your database if you see 2 user documents, 3 street art documents and 3 visit documents. You can use MongoDB Compass for this**

![Imgur](https://i.imgur.com/47GAyyM.png)


### Iteration 3 | Backend | `GET /api/street-arts`

The goal of this iteration is to create a backend route `GET /api/street-arts` where you can access all street arts.

First, update the file `server/app.js`

```js
// server/app.js

// ...

app.use('/api', require('./routes/index'))
app.use('/api', require('./routes/auth'))
app.use('/api/street-arts', require('./routes/street-arts')) // NEW LINE: take all the routes defined in './routes/street-arts' and prefix them by '/api/street-arts'

// ...
```

Then, create a file `server/routes/street-arts.js` and write the necessary code for the route.

```js
// server/routes/street-arts.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res, next) => {
  // TODO
});

module.exports = router;
```


When you are done, you can test with Postman `GET http://localhost:5000/api/street-arts`


![Imgur](https://i.imgur.com/oSDgvgS.png)


### Iteration 4 | Backend | `GET /api/street-arts/:streetArtId`

For this iteration, you have to do something very similar and create to route `GET /api/street-arts/:streetArtId`.

You can create it directly in the file `server/routes/street-arts.js`.


To make sure your route is working, you should test it with Postman!
![Imgur](https://i.imgur.com/V1tMjG1.png)


### Iteration 5 | Backend | `POST /api/street-arts`

In the route `POST /api/street-arts`, you will create a new street art by send 3 informations:
- `lat`: The latitude
- `lng`: The longitude
- `picture`: A file with the picture

To save the file, you will rely on Cloudinary.

For this, you have to install some packages:
```sh
$ cd server
$ npm install cloudinary multer-storage-cloudinary multer
```

Then, you have to go on Cloudinary and find your credentials on the main page: https://cloudinary.com/console. You have to past the crendentials in `server/.env` file.

```
PORT=5000
SESSION_SECRET=anyValue
MONGODB_URI=mongodb://localhost/mern-street-art
CLOUDINARY_NAME=......
CLOUDINARY_KEY=......
CLOUDINARY_SECRET=......
```

Then, you can create a file `server/configs/cloudinary.js`

```js
// server/configs/cloudinary.js
const cloudinary = require('cloudinary');
const cloudinaryStorage = require('multer-storage-cloudinary')
const multer = require('multer');

cloudinary.config({
  cloud_name: process.env.CLOUDINARY_NAME,
  api_key: process.env.CLOUDINARY_KEY,
  api_secret: process.env.CLOUDINARY_SECRET
});

const storage = cloudinaryStorage({
  cloudinary: cloudinary,
  folder: 'street-art-pictures',
  allowedFormats: ['jpg', 'png'],
  filename: (req, file, cb) => {
    cb(null, file.originalname); // The file on cloudinary would have the same name as the original file name
  }
});

const uploader = multer({ storage });
module.exports = uploader;
```

Then, you have to create a route in `server/routes/street-arts.js`
```js
// Route to create a street art
// `uploader.single('picture')` parses the data send with the name `picture` and save information inside `req.file`
router.post('/', uploader.single('picture'), (req, res, next) => {
  let { lat, lng } = req.body
  let pictureUrl = req.file.url
  
  // TODO: continue
  // ...
});
```

When you are done, you can try with Postman. You will have to send the data with "`Body` > `form-data`" and choose type "`File`" for the `picture`.

![](https://i.imgur.com/aqwobhl.png)

After doing the request in Postman, you should see a new document with the information you wrote!


![Imgur](https://i.imgur.com/gVhtsxK.png)



### Iteration 6 | Backend | Fix the signup and login

If you inspect the project, inside `server/routes/auth.js` you already have 2 routes for signup (`POST /api/signup`) and login (`POST /api/login`). The problem with them is that they rely on 2 fields, "username"/"password", instead of "email"/"password".

Let's change this!

Go to `server/routes/auth.js` and find and replace all `username` by `email`.

To make the code cleaner, in the signup route, you can also remove everything related `name` (you don't need the name of the user in this project).

```js
// server/routes/auth.js

// Around line 11
const { email, password } = req.body

// Around line 24
const newUser = new User({ email, password: hashPass })
```

Now it's time to test!

**`POST /api/signup` with Postman**
![Imgur](https://i.imgur.com/BUsmmzx.png)

**`POST /api/login` with Postman**
![Imgur](https://i.imgur.com/2G0tuLk.png)

**Verification with MongoDB Compass**
![](https://i.imgur.com/q04ki4z.png)


### Iteration 7 | Backend | `GET /api/my-visits`

The goal of the route `GET /api/my-visits` is to display all the visits of the connected user, with informations of the street-art.

First, you have to create a file `server/routes/visits.js`
```js
const express = require('express');
const StreetArt = require('../models/StreetArt');
const Visit = require('../models/Visit');
const { isLoggedIn } = require('../middlewares')
const router = express.Router();

// Route protected for logged in user
router.get('/my-visits', isLoggedIn, (req, res, next) => {
  // TODO: continue
  // You should use `.populate`
});

module.exports = router;
```


Then, add the following line in `server/app.js`: 
```js
app.use('/api', require('./routes/visits'))
```

Now, it's time to test with Postman. Be careful, you need to be connected to test.

**`POST /api/login`**: to connect the user
![Imgur](https://i.imgur.com/26pPitg.png)


**`GET /api/my-vistits`**: to get all the visits with the information of the street-arts. Make sure you only have the visits of the connected user.
![Imgur](https://i.imgur.com/VRnit8T.png)


### Iteration 8 | Backend | `POST /api/visits`

Create a route `POST /api/visits` (with a `_streetArt` field) that creates a new visit for the connected user.

This is what you should see with Postman when you are done.
![Imgur](https://i.imgur.com/eocXrj2.png)


### Iteration 9 | Backend | `DELETE /api/visits/:visitId`

Create a route `DELETE /api/visits/:visitId` that deletes the specified visit. 

⚠️ Be careful, only the owner of the visit can delete his visit.

This is what you should see with Postman when you are done.
![Imgur](https://i.imgur.com/fnTtbFy.png)

**NOTE**: We recommand you to make sure that a connected user can't delete the visit of another user.


### Iteration 10 | Frontend | Add of Bootstrap + Reactstrap + `MainNavbar` 

Now it's time to start the front-end part 🔥

In this iteration, you will just add [Bootstrap](https://getbootstrap.com) with [Reactstrap](http://reactstrap.github.io).

For this, you have to install 2 packages:
```sh
$ cd client
$ npm install bootstrap reactstrap
``` 

Then, add the following line in your `client/src/index.scss`:
```scss
@import '../node_modules/bootstrap/scss/bootstrap.scss';
```

Then you can create a navbar in the file `client/src/MainNavbar.jsx`.

The Navbar contains the following links:
- `/` (logo on the left)
- `/list`
- `/map`
- `/new-street-art`
- `/signup`
- `/login`


![Imgur](https://i.imgur.com/NHPUi5Y.png)


### Iteration 11 | Frontend | Simple page component `List`

The goal of this iteration is to create a simple page "/list".

First, in `client/src/api.js`, you have to create a method `getStreetArts`:
```js
// client/src/api.js
// ...

  getStreetArts() {
    return service
      .get('/street-arts')
      .then(res => res.data)
      .catch(errHandler)
  },
```


Then, in `client/src/components/App.jsx`, add a `<Route />` for a `List` component.


Finally, you have to create the `List` component in `client/src/components/pages/List.jsx`. It should display a table with all street arts from the backend, thanks to `api.getStreetArts()`.

Your table will have 3 columns at this stage:
- A small version of the picture
- The coordinates with a link to Google Maps. Example of a link: https://www.google.com/maps/dir//38.69606,-9.209744/@38.69606,-9.209744,15z
- A button link to the future detail page. Example of a link: http://localhost:3000/street-art-detail/5cc046767f07704e3d51c64f


You can preview the page here: 

![Imgur](https://i.imgur.com/m6J0OOb.png)


### Iteration 12 | Frontend | Simple page component `StreetArtDetail`

The goal of this iteration is to create a simple page "/street-art-detail/:streetArtId".

First, in `client/src/api.js`, you have to create a method `getStreetArt`:
```js
// client/src/api.js
// ...

  getStreetArt(streetArtId) {
    // TODO
  },
```

Then, in `client/src/components/App.jsx`, add a `<Route />` for a `StreetArtDetail` component.

Finally, you have to create the `StreetArtDetail` component in `client/src/components/pages/StreetArtDetail.jsx`. You should display the picture and the coordinates of the street art.

You can preview the page here: 
![Imgur](https://i.imgur.com/ySZfLm1.jpg)


**BONUS**: Display the picture fullscreen when the user clicks on the picture and go back to the original view when the user clicks again.


### Iteration 13 | Frontend | Add a map in `StreetArtDetail`

You are going to include a map from Mapbox for this iteration. For this, you need to create an account on [Mapbox](https://mapbox.com) and then follow the next instructions.

```sh
$ cd client
$ npm install mapbox-gl
```

```scss
// client/src/index.scss
@import '../node_modules/mapbox-gl/src/css/mapbox-gl.css';
```

```js
// client/src/components/pages/StreetArtDetail.jsx
// ...

import mapboxgl from 'mapbox-gl/dist/mapbox-gl' // NEW

// Inform your Mapbox token (https://www.mapbox.com/account/)
mapboxgl.accessToken = 'YourToken' // NEW

export default function StreetArtDetail(props) {
  const streetArtId = props.match.params.id
  const mapDomRef = useRef(null) // NEW
  let map = useRef(null).current // NEW
  let marker = useRef(null).current // NEW

  const [streetArt, setStreetArt] = useState(null)
  useEffect(() => {
    api.getStreetArt(streetArtId).then(streetArt => {
      setStreetArt(streetArt)
      let [lng, lat] = streetArt.location.coordinates // NEW
      initMap(lng, lat) // NEW
    })
  }, [])

  function initMap(lng, lat) {
    // NEW METHOD
    // Embed the map where "mapDomRef" is defined in the render
    map = new mapboxgl.Map({
      container: mapDomRef.current,
      style: 'mapbox://styles/mapbox/streets-v11',
      center: [lng, lat],
      zoom: 10,
    })

    // Add zoom control on the top right corner
    map.addControl(new mapboxgl.NavigationControl())

    // Create a marker on the map with the coordinates ([lng, lat])
    marker = new mapboxgl.Marker({ color: 'red' })
      .setLngLat([lng, lat])
      .addTo(map)
  }

  render() {
    // ...
      <div ref={mapDomRef} style={{height: 400}}></div> {/* NEW */}
    // ...
  }
}
```

You can preview the page here: 
![Imgur](https://i.imgur.com/IuZMhjF.jpg)

**NOTE**: If the map is only half displayed, change some CSS to make sure your map is not in a `text-align: center`.


### Iteration 14 | Frontend | Page component `NewStreetArt`

For this iteration, you need to:
- Create a method `addStreetArt(uploadData)` in `client/src/api.js`
- Update `client/src/components/App.jsx` to include a new `<Route />`
- Create a component `NewStreetArt` saved in `client/src/components/pages/NewStreetArt.jsx`


**`client/src/api.js`**
```js
// ... 
addStreetArt(uploadData) {
  return service
    .post('/street-arts', uploadData)
    .then(res => res.data)
    .catch(errHandler)
},
// ... 
```

**`client/src/components/pages/NewStreetArt.jsx`**
```js
import React, { Component } from 'react'
import {
  Button,
  Col,
  Container,
  Input,
  Label,
  Row
} from 'reactstrap'
import api from '../../api'

export default class NewStreetArt extends Component {
  constructor(props) {
    super(props)
    this.state = {
      lat: '',
      lng: '',
      picture: null
    }
    this.handleInputChange = this.handleInputChange.bind(this)
    this.handleFileChange = this.handleFileChange.bind(this)
    this.addStreetArtAndRedirectToDetailPage = this.addStreetArtAndRedirectToDetailPage.bind(this)
    this.getCurrentCoordinates = this.getCurrentCoordinates.bind(this)
  }
  getCurrentCoordinates() {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(position => {
        console.log("The current coords are", position.coords)
        this.setState({
          lng: "TODO", // TODO: write the correct value
          lat: "TODO" // TODO: write the correct value
        })
      })
    }
  }
  handleInputChange(e) {
    this.setState({
      [e.target.name]: e.target.value
    })
  }
  handleFileChange(e) {
    console.log("The file added by the use is: ", e.target.files[0])
    this.setState({
      picture: e.target.files[0]
    })
  }
  addStreetArtAndRedirectToDetailPage(e) {
    // To send information with "form-data" (like in Postman)
    const uploadData = new FormData()
    uploadData.append("lng", this.state.lng)
    uploadData.append("lat", this.state.lat)
    uploadData.append("picture", this.state.picture)

    api.addStreetArt(uploadData)
      .then(createdStreetArt => {
        // Redirect the user to another page
        this.props.history.push('/todo') // TODO
      })
      .catch(err => {
        console.log("Error while adding the street art: ", err)
      })
  }
  render() {
    return (
      <Container className="NewStreetArt">
        <h1>New Street Art</h1>

        <Button className="my-4" color="danger" block outline onClick={this.getCurrentCoordinates}>
          Get Current Coordinates
        </Button>

        <Row className="my-4">
          <Col sm={3}>
            <Label for="exampleEmail">Coordinates</Label>
          </Col>
          <Col>
            <Input type="number" value={this.state.lng} onChange={this.handleInputChange} name="lng" placeholder="Longitude" />
          </Col>
          <Col>
            <Input type="number" value={this.state.lat} onChange={this.handleInputChange} name="lat" placeholder="Latitude" />
          </Col>
        </Row>

        <Row className="my-4">
          <Col sm={3}>
            <Label for="exampleEmail">Picture</Label>
          </Col>
          <Col>
            <Input type="file" name="picture" onChange={this.handleFileChange} />
          </Col>
        </Row>

        <Button className="my-4" color="danger" block onClick={this.addStreetArtAndRedirectToDetailPage}>
          Add Street Art
        </Button>

      </Container>
    )
  }
}

```

You can preview the page here: 
![Imgur](https://i.imgur.com/oeNcorX.png)



### Next iterations
- Iteration x | Frontend | Simple page component `Map`
- Iteration x | Frontend | Change of signup and login
- Iteration x | Frontend | Add of visits in `List`
- Iteration x | Frontend | Add of visits in `StreetArtDetail`
- Iteration x | Frontend | Add of visits in `Map`
- Iteration x | Deploy 🚀
