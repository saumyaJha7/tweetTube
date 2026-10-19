# TweetTube

A YouTube + Twitter style backend built with **Node.js**, **Express**, **MongoDB**, and **Cloudinary**. It supports user authentication, video publishing, tweets, comments, likes, playlists, subscriptions, and a creator dashboard.

## Tech Stack

- **Runtime:** Node.js (ES Modules)
- **Framework:** Express 5
- **Database:** MongoDB with Mongoose
- **Authentication:** JWT (access + refresh tokens) stored in HTTP-only cookies
- **File Storage:** Cloudinary (videos, thumbnails, avatars, cover images) via Multer
- **Pagination:** `mongoose-aggregate-paginate-v2`

## Features

- User registration and login with JWT access/refresh tokens
- Update account details, avatar, cover image, and password
- Channel profiles with subscriber counts
- Watch history
- Video upload, update, delete, and publish toggle
- Tweets (create, read, update, delete)
- Comments with pagination
- Likes on videos, comments, and tweets
- Playlists with add/remove video support
- Channel subscriptions
- Creator dashboard (videos and stats)
- Health check endpoint

## Project Structure

```
src/
├── controllers/     # Route handlers / business logic
├── db/              # MongoDB connection
├── middlewares/     # Auth (JWT) and Multer upload middleware
├── models/          # Mongoose schemas
├── routes/          # Express routers
├── utils/           # ApiError, ApiResponse, asyncHandler, Cloudinary helpers
├── app.js           # Express app configuration and route mounting
├── constants.js     # Database name constant
└── index.js         # App entry point
public/
└── temp/            # Temporary local storage for uploads
```

## Getting Started

### Prerequisites

- Node.js 18+
- MongoDB (local or Atlas)
- Cloudinary account

### Installation

```bash
git clone <repository-url>
cd 03_megaProject
npm install
```

### Environment Variables

Create a `.env` file in the project root:

```env
PORT=4000

MONGODB_URI=mongodb+srv://<user>:<password>@cluster.mongodb.net
DB_USERNAME=<mongodb-username>
DB_PASSWORD=<mongodb-password>

CORS_ORIGIN=http://localhost:5173

ACCESS_TOKEN_SECRET=<random-secret>
ACCESS_TOKEN_EXPIRY=1d
REFRESH_TOKEN_SECRET=<random-secret>
REFRESH_TOKEN_EXPIRY=10d

CLOUDINARY_CLOUD_NAME=<cloud-name>
CLOUDINARY_API_KEY=<api-key>
CLOUDINARY_API_SECRET=<api-secret>
CLOUDINARY_URL=cloudinary://<api-key>:<api-secret>@<cloud-name>
```

### Run the Server

```bash
# development (nodemon)
npm run dev
```

The server runs at `http://localhost:4000` by default.

## API Reference

Base URL: `/api/v1`

### Health Check

| Method | Endpoint        | Description         |
| ------ | --------------- | ------------------- |
| GET    | `/healthCheck`  | Check server health |

### Users

| Method | Endpoint                   | Auth | Description                     |
| ------ | -------------------------- | ---- | ------------------------------- |
| POST   | `/users/register`          | No   | Register (multipart: `avatar`, `coverImage`) |
| POST   | `/users/login`             | No   | Login with email and password   |
| POST   | `/users/logout`            | Yes  | Logout                          |
| POST   | `/users/refresh-token`     | No   | Refresh access token            |
| POST   | `/users/change-password`   | Yes  | Change current password         |
| GET    | `/users/current-user`      | Yes  | Get current user                |
| PATCH  | `/users/update-account`    | Yes  | Update full name and email      |
| PATCH  | `/users/update-avatar`     | Yes  | Update avatar (multipart)       |
| PATCH  | `/users/update-cover-image`| Yes  | Update cover image (multipart)  |
| GET    | `/users/channel/:username` | Yes  | Get channel profile             |
| GET    | `/users/watch-history`     | Yes  | Get watch history               |

### Videos

| Method | Endpoint                        | Auth | Description                             |
| ------ | ------------------------------- | ---- | --------------------------------------- |
| GET    | `/videos`                       | Yes  | Get all videos                          |
| POST   | `/videos`                       | Yes  | Publish video (multipart: `videoFile`, `thumbnail`) |
| GET    | `/videos/:videoId`              | Yes  | Get a video by ID                       |
| PATCH  | `/videos/:videoId`              | Yes  | Update video (multipart: `thumbnail`)   |
| DELETE | `/videos/:videoId`              | Yes  | Delete a video                          |
| PATCH  | `/videos/toggle/publish/:videoId` | Yes | Toggle publish status                   |

### Tweets

| Method | Endpoint               | Auth | Description          |
| ------ | ---------------------- | ---- | -------------------- |
| POST   | `/tweets`              | Yes  | Create a tweet       |
| GET    | `/tweets/user/:userId` | Yes  | Get a user's tweets  |
| PATCH  | `/tweets/:tweetId`     | Yes  | Update a tweet       |
| DELETE | `/tweets/:tweetId`     | Yes  | Delete a tweet       |

### Comments

| Method | Endpoint                   | Auth | Description                          |
| ------ | -------------------------- | ---- | ------------------------------------ |
| GET    | `/comments/:videoId`       | Yes  | Get comments (query: `page`, `limit`) |
| POST   | `/comments/:videoId`       | Yes  | Add a comment                        |
| PATCH  | `/comments/c/:commentId`   | Yes  | Update a comment                     |
| DELETE | `/comments/c/:commentId`   | Yes  | Delete a comment                     |

### Likes

| Method | Endpoint                    | Auth | Description             |
| ------ | --------------------------- | ---- | ----------------------- |
| POST   | `/likes/toggle/v/:videoId`  | Yes  | Toggle video like       |
| POST   | `/likes/toggle/c/:commentId`| Yes  | Toggle comment like     |
| POST   | `/likes/toggle/t/:tweetId`  | Yes  | Toggle tweet like       |
| GET    | `/likes/videos`             | Yes  | Get liked videos        |

### Playlists

| Method | Endpoint                                | Auth | Description                    |
| ------ | --------------------------------------- | ---- | ------------------------------ |
| POST   | `/playlist`                             | Yes  | Create a playlist              |
| GET    | `/playlist/:playlistId`                 | Yes  | Get a playlist by ID           |
| PATCH  | `/playlist/:playlistId`                 | Yes  | Update a playlist              |
| DELETE | `/playlist/:playlistId`                 | Yes  | Delete a playlist              |
| PATCH  | `/playlist/add/:videoId/:playlistId`    | Yes  | Add video to playlist          |
| PATCH  | `/playlist/remove/:videoId/:playlistId` | Yes  | Remove video from playlist     |
| GET    | `/playlist/user/:userId`                | Yes  | Get playlists of a user        |

### Subscriptions

| Method | Endpoint                        | Auth | Description                     |
| ------ | ------------------------------- | ---- | ------------------------------- |
| POST   | `/subscriptions/c/:channelId`   | Yes  | Toggle subscription             |
| GET    | `/subscriptions/c/:channelId`   | Yes  | Get subscribers of a channel    |
| GET    | `/subscriptions/u/:subscriberId`| Yes  | Get channels a user subscribed to |

### Dashboard

| Method | Endpoint             | Auth | Description               |
| ------ | -------------------- | ---- | ------------------------- |
| GET    | `/dashboard/stats`   | Yes  | Get channel stats         |
| GET    | `/dashboard/videos`  | Yes  | Get channel videos        |

## Authentication

Protected routes require a valid access token. The server reads the token from:

1. The `accessToken` HTTP-only cookie (set on login), or
2. The `Authorization` header as `Bearer <token>`.

Refresh tokens are stored on the user document and rotated on every `/users/refresh-token` request.

## Response Format

Successful responses are returned through `ApiResponse`:

```json
{
  "success": true,
  "statusCode": 200,
  "message": "Message",
  "data": {}
}
```

Errors are thrown via `ApiError` and contain `success: false`, `statusCode`, `message`, and optional `errors`.

## Notes

- Uploaded files are temporarily saved to `public/temp`, uploaded to Cloudinary, then deleted locally.
- `getAllVideos` and `getChannelStats` are still under development.
- Do not commit your `.env` file; it contains secrets.
