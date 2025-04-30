# Cat Database Microservice

This microservice manages **cat listing data**, **geolocation**, and **image
uploads** for the **CatCall** application. It supports cat filtering,
location-based search, and file handling for cat profile images.

## Notes

- This service is designed to run as part of the
  [CatCall application](https://github.com/zandonella/CatCall) and is not
  intended for standalone production use.
- MongoDB must be running and accessible using the URI defined in your `.env`
  file.
- Uploaded images are stored in the local filesystem and served statically from
  `/uploads`.

## Base Configuration

- **Base URL:** `http://localhost:<PORT>/api`
- **Default Port:** `3000`
- **Port in CatCall:** Determined by `PORT_<SERVICE>` in the root `.env`
- **Change the Port:** Set `PORT=<your_port>` in a `.env` file (see
  `.env.example`)
- **Content Type:** `application/json`
- **Response Format:** `JSON`

---

## Endpoints

> All endpoints expect and return JSON. If a server or database error occurs, a
> `500 Internal Server Error` will be returned with a generic error message.

| Method | Route                         | Description                              |
| ------ | ----------------------------- | ---------------------------------------- |
| POST   | `/api/uploadCat`              | Upload a new cat profile and image       |
| GET    | `/api/cats`                   | Retrieve all cats, supports filtering    |
| GET    | `/api/cats/:id`               | Get a specific cat by UUID               |
| GET    | `/api/catImage/:owner/:catID` | Return the image path for a specific cat |
| PUT    | `/api/cats/:id`               | Update a cat’s profile fields            |
| DELETE | `/api/cats/:id`               | Delete a cat profile by UUID             |

---

### `POST /api/uploadCat`

**Upload a new cat profile along with an image.**

- Accepts a `multipart/form-data` body with fields:
  - `name`, `age`, `sex`, `breed`, `color`, `owner`, `city`, `state` (all
    required)
  - `image`: one image file
- Automatically assigns a UUID, geocodes city/state to lat/lon, and saves the
  image under `/uploads/<owner>/<uuid>.ext`

**Success Response:**

```json
{ "message": "Cat uploaded" }
```

**Error Response:**

```json
{ "error": "Missing cat data" }
```

---

### `GET /api/cats`

**Retrieve all cats, with optional filters.**

Supported query parameters:

- `owner`, `color`, `sex`, `breed`, `minAge`, `maxAge`
- `lat`, `lon`, `radius` — enables geolocation-based filtering

**Example:**

```http
GET /api/cats?color=black&minAge=2&lat=44.5645659.5&lon=-123.2620435&radius=30
```

> Searches for all cats in the database that are black and at least 2 years old
> within a 30 mile radius of Corvallis, Oregon.

**Response:**

```json
{
  "cats": [ { "_id": "...", "name": "...", ... } ]
}
```

---

### `GET /api/cats/:id`

**Retrieve a specific cat profile by UUID.**

**Success:**

```json
{
  "_id": "...",
  "name": "...",
  "age": 3,
  ...
}
```

**Error:**

```json
{ "error": "Cat not found" }
```

---

### `GET /api/catImage/:owner/:catID`

**Returns the image file for a specific cat ID and owner.**

Returns the static file path or:

```json
{ "error": "Image not found" }
```

---

### `PUT /api/cats/:id`

**Update a cat profile’s text fields.**

**Request Body:**

```json
{
  "name": "Updated Name",
  "age": 4,
  "sex": "Female",
  "breed": "Calico",
  "color": "White"
}
```

**Success:**

```json
{ "message": "Cat updated", "updatedCat": { ... } }
```

**Error:**

```json
{ "error": "Cat not found" }
```

---

### `DELETE /api/cats/:id`

**Delete a cat profile by UUID.**

**Success:**

```json
{ "message": "Cat deleted successfully" }
```

**Error:**

```json
{ "error": "Cat not found" }
```

---

## Environment Setup To Run Locally

1. Copy the example environment file:

```bash
cp .env.example .env
```

2. Modify the values in `.env` as needed:

**Example `.env` contents:**

```env
PORT=3000
MONGO_URI=mongodb://localhost:27017
```

---

## Running Locally (Without Docker)

Make sure [Node.js](https://nodejs.org/) is installed and MongoDB is running.

```bash
npm install
npm start
```

Expected output:

```
Connected to MongoDB successfully
Server is running on port 3000.
```

---

## Running with Docker

You can also run this microservice in isolation using Docker:

### 1. Build the image

```bash
docker build -t cat-database-microservice .
```

### 2. Run the container

```bash
docker run -p 3000:3000 --env-file .env cat-database-microservice
```

> ⚠️ Ensure your `.env` file is in the root and includes `MONGO_URI`.

---
