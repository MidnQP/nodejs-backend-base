<p align=center>
	<img heigh=auto width=400 src="https://cdn.pixabay.com/photo/2015/04/23/17/41/node-js-736399_960_720.png"/>
	<p align=center><b>D E V &nbsp;&nbsp;&nbsp; G U I D E L I N E S</b></p>
</p>
<!--hr></hr-->

<img height=450px width=auto src=https://user-images.githubusercontent.com/90899789/177304942-f50b8073-cd1c-4e18-9f6c-281903d4a710.gif />


The purpose of the guidelines are:
- to bring consistency & uniformity in Node.js backend code
- to enable developers to focus on the business value, instead of the app itself
- to enable developers to write code faster, that are:
	- clean
	- readable & maintainable
	- scalable

Some files are added as examples in the project structure. Noteworthy that, this project is intended for a monolith.

### Features

1. Simple & scalable project structure
2. Consistent dev environment
	- precommit hooks to format code, for uniformity in code style
	- support for multiple platform i.e. Windows, Mac, Linux
3. TypeScript-integrated ORM, Sequelize
4. TypeScript-integrated validator, Joi
5. TypeScript-integrated tests for API response compatibility, to check and fail tests, if there are:
	- changes in properties' data-type
	- new properties
	- removed properties
	- null-able properties
6. Built-in DevMail, to alert developers when backend crashed in production
7. `TODO` Built-in support for file upload
8. `TODO` Built-in security features

### Structure
```
	.
	├── server.ts (the main file)
	├── app.ts    (the application)
	├── bin/      (various scripts)
	├── test/
	└── src/
		├── configs/
		├── constants/
		├── db/            (database models)
		├── dto/           (request data validations)
		├── migrations/    (database migrations)
		├── utils/         (helper utils)
		└── services/      (business domains)
		    ├── index.ts   (exports all apis & service methods)
		    ├── order/
		    ├── shop/
		    └── user/
```

### Service Scalability

Let's take from examples. 

Firstly, let's say we want to add an Item service to our application. The admin will manage the items. So, it's going to be simple CRUD APIs, storing data into DB. To implement, we'll create a simple file with 5 standard methods in `src/services/item/index.ts`:
```ts
export default class Item {
	static create(req, rest) {}
	static update(req, res) {}
	static list(req, res) {}
	static get(req, res) {}
	static remove(req, res) {}
}
```

Secondly, let's say we want to add a Cart service to our application. Now, this is relatively complicated than Item service. Lots of methods & business logic, security, and performance to consider. In this case, we'd create files like below:
```
    services/
    └── cart/
	    ├── index.ts
	    ├── add.ts
	    ├── update.ts
	    ├── checkout.ts
	    ├── free-campaign.ts
	    ├── calculate.ts
	    └── list.ts
```

## REST
One of the most important guiding principles of REST is _stateless_. 
Meaning, the requests do not reuse any previous context. Each request
contains enough info to understand the individual request.

To learn in details about API design, every backend developer should read the [Google Cloud API Design](https://cloud.google.com/apis/design) guide.

### Concepts
In REST, the primary data representation is called a "resource".

- A **collection resource** (plural naming), identified by URI `/users`.
- A single **document resource** (singular naming), identified by URI `/users/:user_id`.
- **Sub-collection resources** are nested. In the URI `/users/:user_id/repositories`, "repositories" is a sub-collection resource. Similarly, a singleton in that sub-collection will be `/users/:user_id/repositories/:repo_id`.


### Consistency
Some constraints in REST API naming ensures a design of scalable API endpoints:
- Use plural nouns to name, and represent resources. Example: `users`, `orders`, `categories`.
- Use hypen, not underscores. Use lowercase letters in URI, never camel-case. `GET /food-categories`, not `GET /foodCategories`.
- No trailing forward slash. Example: `GET /users/` is wrong, and `GET /users` is correct.
- Never use CRUD function names in URIs, like `/users/list` or `/users/create`. Rather use:
	- `GET /users` - get all users.
	- `GET /users/:id` - get a single user.
	- `POST /users/:id` - create a single user.
	- `PUT /users/:id` - update a single user.
	- `DELETE /users/:id` - delete a single user.
- Use query components to filter collection, never use for anything else. Example: `/users?region=Malaysia&sort=createdAt`.


## Data Objects

### Client Data

Joi is used to validate/sanitize the client data. JSON is primarily supported.
The purpose of request fields are:
- `req.body`: 
	- To send data in POST and PUT
	- To create/update. 
	- In many platforms, in a GET request, the request body is ignored/removed. Example: Swagger.
- `req.query`: 
	- To control what data is returned in endpoint responses.
	- To sort.
	- To filter.
	- To add search condition.
	- Example: `/users?region=Malaysia&sort=createdAt`.
- `req.params`:
	- Path variables are used to get a singleton from a collection resource.
	- Example: `GET /users/{user_id}`, `GET /categories/{id}`.

To get type-annotated & type-safe DTO:
```ts
//// file: src/dto/user.ts
const signup = {
	body : joi.object({
		userType  : joi.string().required().valid(userTypeList),
		password  : joi.string().password.required(),
		mobile    : joi.string().optional(),
		firstName : joi.string().required(),
		email     : joi.string().email().required(),
	}).required(),
}

//// file: src/modules/user/index.ts
import {Request} from 'express'
import dto from '@src/dto'

function signup (req:Request, res) {
	const data = req.data(dto.user.signup, 'body')
	data.email = 123 // TS ERROR, since TS knows that `email` is `string`.
}
```

### Data Model

Sequelize is the most popular Node.js ORM in the entire NPM registry, Mongoose being an ODM. Sequelize and Prisma compete closely, so they are almost the same popular. However, much of the features of Sequlize requires dynamic configuration internally. So static analysis doesn't become possible, and Sequelize doesn't provide much advantage of type-anotation with TypeScript. That's why efforts were undertaken in this regard.

To get type-annotated & type-safe DAO:
```ts
//// file: src/db/user.ts
@Table
export default class user extends Model<attr, crAttr> {
	@PrimaryKey
	@Column(DataType.UUIDV4)
	id!: CreationOptional<string>

	@Column
	name!: string

	@Column
	email!: string
}

//// file: src/services/user/index.ts
import db from '@src/db'

async function login (req, res) {
	const user = await db.user.findById(10)
	user.email = 123 // TS ERROR, since TS knows that `email` is `string`.
	user.save()
}
```
