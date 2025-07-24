# ArcadeForge API Documentation

This document provides comprehensive API documentation for the ArcadeForge game creation platform.

## Overview

The ArcadeForge API is a RESTful API built with Fastify, providing endpoints for game management, asset handling, user authentication, and real-time collaboration. All API responses return JSON data and use standard HTTP status codes.

**Base URL**: `https://api.arcadeforge.com/v1`

## Authentication

### JWT Bearer Token
All authenticated endpoints require a JWT bearer token in the Authorization header:

```http
Authorization: Bearer <your-jwt-token>
```

### OAuth Login
```http
POST /auth/oauth/google
Content-Type: application/json

{
  "code": "google-oauth-code",
  "redirect_uri": "https://app.arcadeforge.com/auth/callback"
}
```

**Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "name": "John Doe",
    "subscription_tier": "pro",
    "created_at": "2024-01-15T10:30:00Z"
  },
  "expires_in": 3600
}
```

## Game Management

### List Games
```http
GET /games
Authorization: Bearer <token>
```

**Query Parameters:**
- `page` (integer): Page number (default: 1)
- `limit` (integer): Items per page (default: 20, max: 100)
- `search` (string): Search game titles
- `sort` (string): Sort field (`created_at`, `updated_at`, `title`)
- `order` (string): Sort order (`asc`, `desc`)

**Response:**
```json
{
  "games": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "title": "Super Platformer Adventure",
      "description": "A classic platformer game with collectibles",
      "thumbnail_url": "https://cdn.arcadeforge.com/thumbnails/game-123.jpg",
      "is_published": true,
      "published_url": "https://play.arcadeforge.com/games/super-platformer-adventure",
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-16T14:22:00Z",
      "stats": {
        "play_count": 1250,
        "like_count": 89,
        "scene_count": 5,
        "asset_count": 23
      }
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 45,
    "total_pages": 3
  }
}
```

### Get Game Details
```http
GET /games/{gameId}
Authorization: Bearer <token>
```

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "Super Platformer Adventure",
  "description": "A classic platformer game with collectibles",
  "game_data": {
    "scenes": [
      {
        "id": "scene-1",
        "name": "Level 1",
        "width": 1920,
        "height": 1080,
        "background_color": "#87CEEB",
        "objects": [
          {
            "id": "player-1",
            "type": "player",
            "x": 100,
            "y": 500,
            "width": 32,
            "height": 48,
            "sprite_id": "player-sprite-1",
            "physics": {
              "has_gravity": true,
              "can_jump": true,
              "max_speed": 200
            }
          }
        ]
      }
    ],
    "settings": {
      "gravity": 980,
      "player_speed": 150,
      "jump_force": 400
    }
  },
  "collaborators": [
    {
      "user_id": "550e8400-e29b-41d4-a716-446655440001",
      "email": "collaborator@example.com",
      "role": "editor",
      "added_at": "2024-01-16T10:00:00Z"
    }
  ],
  "version": 15,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-16T14:22:00Z"
}
```

### Create Game
```http
POST /games
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "My New Game",
  "description": "A platformer game",
  "template": "platformer" // Optional: "platformer", "puzzle", "shooter", "blank"
}
```

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440002",
  "title": "My New Game",
  "description": "A platformer game",
  "game_data": {
    "scenes": [
      {
        "id": "scene-1",
        "name": "Scene 1",
        "width": 1920,
        "height": 1080,
        "objects": []
      }
    ]
  },
  "version": 1,
  "created_at": "2024-01-17T09:15:00Z"
}
```

### Update Game
```http
PUT /games/{gameId}
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "Updated Game Title",
  "description": "Updated description",
  "game_data": {
    "scenes": [...],
    "settings": {...}
  }
}
```

### Delete Game
```http
DELETE /games/{gameId}
Authorization: Bearer <token>
```

### Publish Game
```http
POST /games/{gameId}/publish
Authorization: Bearer <token>
Content-Type: application/json

{
  "is_public": true,
  "custom_url": "my-awesome-game" // Optional custom URL slug
}
```

**Response:**
```json
{
  "published_url": "https://play.arcadeforge.com/games/my-awesome-game",
  "embed_code": "<iframe src=\"https://play.arcadeforge.com/embed/my-awesome-game\" width=\"800\" height=\"600\"></iframe>",
  "published_at": "2024-01-17T12:00:00Z"
}
```

## Asset Management

### Upload Asset
```http
POST /assets
Authorization: Bearer <token>
Content-Type: multipart/form-data

file: <binary-file-data>
name: "player-sprite.png"
tags: "character,player,sprite"
```

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440003",
  "filename": "player-sprite.png",
  "original_name": "player-sprite.png",
  "file_type": "image/png",
  "file_size": 15420,
  "url": "https://cdn.arcadeforge.com/assets/550e8400-e29b-41d4-a716-446655440003.png",
  "thumbnail_url": "https://cdn.arcadeforge.com/thumbnails/550e8400-e29b-41d4-a716-446655440003.jpg",
  "metadata": {
    "width": 64,
    "height": 64,
    "format": "PNG",
    "has_transparency": true
  },
  "tags": ["character", "player", "sprite"],
  "created_at": "2024-01-17T11:30:00Z"
}
```

### List Assets
```http
GET /assets
Authorization: Bearer <token>
```

**Query Parameters:**
- `page`, `limit`: Pagination
- `type`: Filter by file type (`image`, `audio`, `video`)
- `tags`: Filter by tags (comma-separated)
- `search`: Search asset names

**Response:**
```json
{
  "assets": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440003",
      "filename": "player-sprite.png",
      "file_type": "image/png",
      "file_size": 15420,
      "url": "https://cdn.arcadeforge.com/assets/550e8400-e29b-41d4-a716-446655440003.png",
      "thumbnail_url": "https://cdn.arcadeforge.com/thumbnails/550e8400-e29b-41d4-a716-446655440003.jpg",
      "tags": ["character", "player", "sprite"],
      "usage_count": 3,
      "created_at": "2024-01-17T11:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 127
  }
}
```

### Get Asset Details
```http
GET /assets/{assetId}
Authorization: Bearer <token>
```

### Update Asset
```http
PUT /assets/{assetId}
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "Updated Asset Name",
  "tags": ["updated", "tags"]
}
```

### Delete Asset
```http
DELETE /assets/{assetId}
Authorization: Bearer <token>
```

### Bulk Asset Operations
```http
POST /assets/bulk
Authorization: Bearer <token>
Content-Type: application/json

{
  "action": "delete", // "delete", "tag", "untag"
  "asset_ids": [
    "550e8400-e29b-41d4-a716-446655440003",
    "550e8400-e29b-41d4-a716-446655440004"
  ],
  "tags": ["archived"] // Only for tag/untag actions
}
```

## User Management

### Get Current User
```http
GET /users/me
Authorization: Bearer <token>
```

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "name": "John Doe",
  "avatar_url": "https://cdn.arcadeforge.com/avatars/user-123.jpg",
  "subscription_tier": "pro",
  "subscription_expires_at": "2024-12-31T23:59:59Z",
  "usage_stats": {
    "games_created": 12,
    "assets_uploaded": 156,
    "storage_used_mb": 245.7,
    "storage_limit_mb": 1000,
    "collaboration_minutes": 1840
  },
  "preferences": {
    "theme": "dark",
    "auto_save_interval": 30,
    "grid_snap": true
  },
  "created_at": "2023-08-15T10:30:00Z"
}
```

### Update User Profile
```http
PUT /users/me
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "Updated Name",
  "preferences": {
    "theme": "light",
    "auto_save_interval": 60
  }
}
```

### Upload Avatar
```http
POST /users/me/avatar
Authorization: Bearer <token>
Content-Type: multipart/form-data

avatar: <image-file-data>
```

## Collaboration

### Get Game Collaborators
```http
GET /games/{gameId}/collaborators
Authorization: Bearer <token>
```

### Add Collaborator
```http
POST /games/{gameId}/collaborators
Authorization: Bearer <token>
Content-Type: application/json

{
  "email": "collaborator@example.com",
  "role": "editor" // "owner", "editor", "viewer"
}
```

### Update Collaborator Role
```http
PUT /games/{gameId}/collaborators/{userId}
Authorization: Bearer <token>
Content-Type: application/json

{
  "role": "viewer"
}
```

### Remove Collaborator
```http
DELETE /games/{gameId}/collaborators/{userId}
Authorization: Bearer <token>
```

## Real-time WebSocket API

### Connection
```javascript
// Connect to WebSocket
const ws = new WebSocket('wss://api.arcadeforge.com/ws');

// Authentication
ws.send(JSON.stringify({
  type: 'auth',
  token: 'your-jwt-token'
}));

// Join game editing session
ws.send(JSON.stringify({
  type: 'join_game',
  game_id: '550e8400-e29b-41d4-a716-446655440000'
}));
```

### Message Types

#### Game Operations
```javascript
// Add/update/delete game objects
ws.send(JSON.stringify({
  type: 'game_operation',
  operation: {
    type: 'add_object',
    scene_id: 'scene-1',
    object: {
      id: 'new-object-1',
      type: 'sprite',
      x: 100,
      y: 200,
      sprite_id: 'player-sprite-1'
    }
  }
}));

// Receive operation from other users
ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  if (message.type === 'game_operation') {
    // Apply operation to local game state
    applyOperation(message.operation);
  }
};
```

#### User Presence
```javascript
// Update cursor position
ws.send(JSON.stringify({
  type: 'cursor_update',
  x: 150,
  y: 300
}));

// Receive cursor updates from other users
ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  if (message.type === 'user_presence') {
    updateUserCursors(message.users);
  }
};
```

#### Selection Sync
```javascript
// Update selected objects
ws.send(JSON.stringify({
  type: 'selection_update',
  selected_objects: ['object-1', 'object-2']
}));
```

## Analytics & Statistics

### Game Analytics
```http
GET /games/{gameId}/analytics
Authorization: Bearer <token>
```

**Query Parameters:**
- `period`: Time period (`7d`, `30d`, `90d`, `1y`)
- `metrics`: Comma-separated metrics (`plays`, `likes`, `shares`, `completion_rate`)

**Response:**
```json
{
  "period": "30d",
  "total_plays": 1250,
  "unique_players": 890,
  "average_play_time": 245,
  "completion_rate": 0.67,
  "daily_stats": [
    {
      "date": "2024-01-01",
      "plays": 45,
      "unique_players": 38,
      "average_play_time": 230
    }
  ],
  "geographic_distribution": {
    "US": 45.2,
    "UK": 18.7,
    "CA": 12.1,
    "DE": 8.9,
    "FR": 6.8,
    "other": 8.3
  }
}
```

### User Analytics
```http
GET /users/me/analytics
Authorization: Bearer <token>
```

## Billing & Subscriptions

### Get Subscription Status
```http
GET /billing/subscription
Authorization: Bearer <token>
```

**Response:**
```json
{
  "tier": "pro",
  "status": "active",
  "current_period_start": "2024-01-01T00:00:00Z",
  "current_period_end": "2024-02-01T00:00:00Z",
  "usage": {
    "games_created": 12,
    "games_limit": 50,
    "storage_used_mb": 245.7,
    "storage_limit_mb": 1000,
    "collaboration_minutes": 1840,
    "collaboration_limit": 10000
  },
  "features": {
    "real_time_collaboration": true,
    "custom_branding": true,
    "analytics": true,
    "priority_support": true
  }
}
```

### Create Checkout Session
```http
POST /billing/checkout
Authorization: Bearer <token>
Content-Type: application/json

{
  "plan": "pro",
  "billing_period": "monthly", // "monthly", "yearly"
  "success_url": "https://app.arcadeforge.com/billing/success",
  "cancel_url": "https://app.arcadeforge.com/billing/cancel"
}
```

## Error Handling

### Standard Error Response
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains invalid data",
    "details": [
      {
        "field": "title",
        "message": "Title is required"
      },
      {
        "field": "game_data.scenes",
        "message": "At least one scene is required"
      }
    ]
  },
  "request_id": "req_550e8400e29b41d4a716446655440000"
}
```

### HTTP Status Codes

- `200 OK`: Successful request
- `201 Created`: Resource created successfully
- `204 No Content`: Successful request with no response body
- `400 Bad Request`: Invalid request data
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Resource not found
- `409 Conflict`: Resource conflict (e.g., duplicate name)
- `413 Payload Too Large`: File upload size exceeded
- `422 Unprocessable Entity`: Valid request format but semantic errors
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server error
- `503 Service Unavailable`: Service temporarily unavailable

### Error Codes

- `VALIDATION_ERROR`: Request validation failed
- `AUTHENTICATION_ERROR`: Invalid or expired token
- `AUTHORIZATION_ERROR`: Insufficient permissions
- `RESOURCE_NOT_FOUND`: Requested resource doesn't exist
- `DUPLICATE_RESOURCE`: Resource already exists
- `RATE_LIMIT_EXCEEDED`: Too many requests
- `FILE_TOO_LARGE`: Upload file size exceeded
- `UNSUPPORTED_FILE_TYPE`: File type not allowed
- `STORAGE_QUOTA_EXCEEDED`: User storage limit reached
- `COLLABORATION_LIMIT_EXCEEDED`: Too many concurrent collaborators
- `PAYMENT_REQUIRED`: Subscription required for this feature

## Rate Limiting

### Limits by Endpoint Type

- **Authentication**: 10 requests per minute
- **Game Operations**: 100 requests per minute
- **Asset Upload**: 20 requests per minute (500MB total per minute)
- **Analytics**: 60 requests per minute
- **General API**: 1000 requests per minute

### Rate Limit Headers
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1642636800
```

## SDK Examples

### JavaScript/TypeScript
```typescript
import { ArcadeForgeAPI } from '@arcadeforge/sdk';

const api = new ArcadeForgeAPI({
  baseURL: 'https://api.arcadeforge.com/v1',
  token: 'your-jwt-token'
});

// Create a new game
const game = await api.games.create({
  title: 'My Game',
  template: 'platformer'
});

// Upload an asset
const asset = await api.assets.upload({
  file: fileBuffer,
  name: 'player-sprite.png',
  tags: ['character', 'player']
});

// Real-time collaboration
const collaboration = api.collaboration.connect(gameId);
collaboration.on('operation', (operation) => {
  // Handle real-time game operations
});
```

### Python
```python
from arcadeforge import ArcadeForgeAPI

api = ArcadeForgeAPI(
    base_url='https://api.arcadeforge.com/v1',
    token='your-jwt-token'
)

# Get user's games
games = api.games.list(limit=50)

# Get game analytics
analytics = api.games.get_analytics(
    game_id='550e8400-e29b-41d4-a716-446655440000',
    period='30d'
)
```

This API documentation provides a comprehensive guide for integrating with the ArcadeForge platform, covering all major functionality needed for game creation, asset management, collaboration, and analytics.