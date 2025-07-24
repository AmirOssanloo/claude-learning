# ArcadeForge Architecture

This document outlines the system architecture for ArcadeForge, a scalable drag-and-drop game creation SaaS platform.

## Overview

ArcadeForge is designed as a distributed system with clear separation of concerns, optimized for performance, scalability, and user experience. The architecture follows hexagonal (ports and adapters) patterns with domain-driven design principles.

## Core Principles

### 1. **Performance First**
- **60fps Target**: All game engine operations must maintain 60fps
- **Progressive Loading**: Games and assets load incrementally
- **Memory Efficiency**: Automatic cleanup and object pooling
- **WebGL Acceleration**: Hardware-accelerated rendering via Pixi.js

### 2. **Scalable Architecture**
- **Microservices**: Independent scaling of game editor, API, and runtime
- **Event-Driven**: Asynchronous communication between services
- **Stateless APIs**: Horizontal scaling without session affinity
- **CDN Integration**: Global asset delivery via CloudFront

### 3. **Real-time Collaboration**
- **Operational Transforms**: Conflict-free concurrent editing
- **WebSocket Communication**: Low-latency bi-directional updates
- **Optimistic Updates**: Immediate UI feedback with server reconciliation
- **Presence Awareness**: Real-time user cursors and selections

## System Architecture

### High-Level Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Game Editor   │    │   API Server    │    │  Game Runtime   │
│   (React/Pixi)  │◄──►│  (Fastify/WS)   │◄──►│   (Pixi.js)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│    Asset CDN    │    │   Database      │    │   Monitoring    │
│  (S3/CloudFront)│    │ (RDS/Redis)     │    │  (CloudWatch)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Component Architecture

#### Game Editor (Frontend)

```typescript
// Main Editor Architecture
apps/game-editor/src/
├── editor/                    # Core editing canvas
│   ├── canvas/               # Pixi.js rendering layer
│   ├── tools/                # Drag-and-drop tools
│   ├── timeline/             # Animation & scene management
│   └── properties/           # Object property panels
├── assets/                   # Asset management
│   ├── upload/               # File upload & processing
│   ├── library/              # Asset organization
│   └── optimization/         # Client-side asset prep
├── collaboration/            # Real-time features
│   ├── cursors/              # User presence indicators
│   ├── operations/           # Operational transforms
│   └── sync/                 # State synchronization
└── preview/                  # Game testing
    ├── player/               # Embedded game runtime
    ├── debugging/            # Developer tools
    └── performance/          # Frame rate monitoring
```

**Key Patterns:**
- **Component Composition**: Reusable editor widgets
- **State Management**: Redux with normalized game state
- **Canvas Rendering**: Pixi.js with WebGL fallback
- **Event System**: Custom event bus for editor actions

#### API Server (Backend)

```typescript
// Hexagonal Architecture Implementation
apps/api-server/src/
├── core/                     # Domain layer (business logic)
│   ├── entities/
│   │   ├── Game.ts           # Game project aggregate
│   │   ├── Asset.ts          # Asset management
│   │   ├── User.ts           # User account
│   │   └── Collaboration.ts  # Real-time session
│   ├── use-cases/
│   │   ├── CreateGame.ts     # Game creation workflow
│   │   ├── ProcessAsset.ts   # Asset upload pipeline
│   │   └── SyncChanges.ts    # Collaboration sync
│   └── ports/                # Interface definitions
│       ├── GameRepository.ts
│       ├── AssetStorage.ts
│       └── NotificationService.ts
├── infrastructure/           # External adapters
│   ├── db/
│   │   ├── PrismaGameRepo.ts # Database implementation
│   │   └── migrations/       # Schema versions
│   ├── storage/
│   │   ├── S3AssetStorage.ts # AWS S3 integration
│   │   └── ImageProcessor.ts # Sharp integration
│   └── cache/
│       └── RedisCache.ts     # Redis implementation
├── framework/                # Fastify setup
│   ├── server/               # Server configuration
│   ├── plugins/              # Auth, validation, etc.
│   └── websockets/           # Real-time communication
└── routes/                   # HTTP/WebSocket handlers
    ├── v1/games/             # Game CRUD operations
    ├── v1/assets/            # Asset management
    ├── v1/users/             # User operations
    └── ws/collaboration/     # WebSocket endpoints
```

**Key Patterns:**
- **Domain-Driven Design**: Rich domain models with business logic
- **CQRS**: Separate read/write models for performance
- **Event Sourcing**: Audit trail for game changes
- **Circuit Breaker**: Resilient external service calls

#### Game Runtime Engine

```typescript
// Optimized Game Player
packages/game-engine/src/
├── core/                     # Engine fundamentals
│   ├── GameLoop.ts           # Main update/render cycle
│   ├── SceneManager.ts       # Scene transitions
│   ├── InputSystem.ts        # Keyboard/mouse/touch
│   └── AssetLoader.ts        # Progressive asset loading
├── rendering/                # Visual systems
│   ├── SpriteRenderer.ts     # 2D sprite rendering
│   ├── AnimationSystem.ts    # Sprite animations
│   ├── ParticleSystem.ts     # Effects & particles
│   └── UIRenderer.ts         # Game UI elements
├── physics/                  # Game mechanics
│   ├── CollisionDetection.ts # AABB & spatial partitioning
│   ├── PlatformerPhysics.ts  # Gravity, jumping, movement
│   └── TriggerSystem.ts      # Interactive game objects
├── audio/                    # Sound management
│   ├── AudioManager.ts       # Howler.js wrapper
│   ├── SpatialAudio.ts       # Positional sound
│   └── MusicSystem.ts        # Background music
└── serialization/            # Game data loading
    ├── GameLoader.ts         # Load game from JSON
    ├── AssetResolver.ts      # Resolve asset references
    └── SceneBuilder.ts       # Construct game scenes
```

**Key Patterns:**
- **Entity-Component-System**: Flexible game object composition
- **Object Pooling**: Memory-efficient sprite management
- **Spatial Partitioning**: Efficient collision detection
- **Asset Streaming**: Progressive loading for large games

## Data Architecture

### Database Design

```sql
-- Core Tables (PostgreSQL)
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    subscription_tier VARCHAR(50),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE games (
    id UUID PRIMARY KEY,
    owner_id UUID REFERENCES users(id),
    title VARCHAR(255) NOT NULL,
    game_data JSONB NOT NULL,  -- Serialized game state
    version INTEGER DEFAULT 1,
    published_url VARCHAR(255),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE assets (
    id UUID PRIMARY KEY,
    owner_id UUID REFERENCES users(id),
    filename VARCHAR(255) NOT NULL,
    file_type VARCHAR(50) NOT NULL,
    file_size INTEGER NOT NULL,
    s3_key VARCHAR(500) NOT NULL,
    metadata JSONB,  -- Dimensions, duration, etc.
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE collaborations (
    id UUID PRIMARY KEY,
    game_id UUID REFERENCES games(id),
    user_id UUID REFERENCES users(id),
    role VARCHAR(50) NOT NULL,  -- owner, editor, viewer
    created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for performance
CREATE INDEX idx_games_owner ON games(owner_id);
CREATE INDEX idx_games_updated ON games(updated_at DESC);
CREATE INDEX idx_assets_owner ON assets(owner_id);
CREATE INDEX idx_collaborations_game ON collaborations(game_id);
```

### Real-time State Management

```typescript
// Redis Data Structures for Collaboration
interface CollaborationSession {
  gameId: string;
  users: {
    [userId: string]: {
      cursor: { x: number; y: number };
      selection: string[];
      lastSeen: number;
    };
  };
  operations: OperationalTransform[];
  version: number;
}

// Operational Transform for Concurrent Editing
interface GameOperation {
  type: 'add' | 'update' | 'delete' | 'move';
  objectId: string;
  data: any;
  userId: string;
  timestamp: number;
  dependencies: string[];  // Causal ordering
}
```

## Scalability Patterns

### Horizontal Scaling

#### API Server Scaling
```yaml
# Auto Scaling Configuration
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-server-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-server
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

#### Database Scaling
- **Read Replicas**: Multiple PostgreSQL read replicas for game data queries
- **Connection Pooling**: PgBouncer for efficient connection management
- **Query Optimization**: Database indexes and query analysis
- **Caching Strategy**: Redis for frequently accessed game data

#### Asset Delivery Scaling
- **S3 Storage**: Virtually unlimited asset storage
- **CloudFront CDN**: Global edge locations for fast asset delivery
- **Progressive JPEG**: Optimized image loading
- **Sprite Atlasing**: Reduced HTTP requests for game assets

### Performance Optimization

#### Frontend Performance
```typescript
// Game Editor Optimizations
class EditorCanvas {
  private renderQueue: RenderOperation[] = [];
  private frameTime = 16.67;  // 60fps target
  
  render() {
    const startTime = performance.now();
    
    // Frustum culling - only render visible objects
    const visibleObjects = this.getVisibleObjects();
    
    // Batch rendering operations
    this.batchRenderOperations(visibleObjects);
    
    // Performance monitoring
    const endTime = performance.now();
    if (endTime - startTime > this.frameTime) {
      console.warn('Frame time exceeded 60fps target');
    }
  }
}
```

#### Backend Performance
```typescript
// API Performance Patterns
class GameService {
  async getGame(gameId: string): Promise<Game> {
    // Check cache first
    const cached = await this.redis.get(`game:${gameId}`);
    if (cached) return JSON.parse(cached);
    
    // Database query with connection pooling
    const game = await this.gameRepo.findById(gameId);
    
    // Cache result with TTL
    await this.redis.setex(`game:${gameId}`, 300, JSON.stringify(game));
    
    return game;
  }
}
```

## Security Architecture

### Authentication & Authorization
- **JWT Tokens**: Stateless authentication with short expiration
- **OAuth Integration**: Google, GitHub social login
- **Role-Based Access**: Owner, Editor, Viewer permissions per game
- **API Rate Limiting**: Per-user request throttling

### Data Security
- **Input Validation**: Zod schemas for all API inputs
- **SQL Injection Prevention**: Prisma ORM parameterized queries
- **XSS Protection**: Content Security Policy headers
- **Asset Scanning**: Malware detection for uploaded files

### Infrastructure Security
- **VPC Isolation**: Private subnets for database and cache
- **WAF Protection**: Web Application Firewall for API endpoints
- **Encryption**: TLS 1.3 for data in transit, AES-256 for data at rest
- **Secrets Management**: AWS Secrets Manager for credentials

## Monitoring & Observability

### Application Metrics
```typescript
// Custom Metrics Collection
class GameMetrics {
  static recordGameCreation(userId: string, gameType: string) {
    metrics.increment('games.created', {
      user_id: userId,
      game_type: gameType
    });
  }
  
  static recordRenderPerformance(frameTime: number) {
    metrics.histogram('editor.frame_time', frameTime);
  }
  
  static recordAssetUpload(fileSize: number, processingTime: number) {
    metrics.histogram('assets.processing_time', processingTime);
    metrics.histogram('assets.file_size', fileSize);
  }
}
```

### Alerting Strategy
- **Error Rate**: Alert on >1% API error rate
- **Response Time**: Alert on >500ms p95 response time
- **Game Performance**: Alert on <30fps average in editor
- **Asset Processing**: Alert on asset processing failures
- **Collaboration Sync**: Alert on WebSocket connection failures

## Deployment Architecture

### AWS Infrastructure
```
┌─────────────────┐
│   CloudFront    │  ←─ Global CDN
│      (CDN)      │
└─────────────────┘
         │
         ▼
┌─────────────────┐
│       ALB       │  ←─ Load Balancer
│ (Load Balancer) │
└─────────────────┘
         │
         ▼
┌─────────────────┐    ┌─────────────────┐
│   ECS Fargate   │    │   ECS Fargate   │  ←─ API Servers
│  (API Servers)  │    │  (API Servers)  │
└─────────────────┘    └─────────────────┘
         │                       │
         ▼                       ▼
┌─────────────────┐    ┌─────────────────┐
│   RDS Postgres  │    │  ElastiCache    │  ←─ Data Layer
│   (Database)    │    │    (Redis)      │
└─────────────────┘    └─────────────────┘
         │                       │
         ▼                       ▼
┌─────────────────┐    ┌─────────────────┐
│       S3        │    │   CloudWatch    │  ←─ Storage & Monitoring
│  (Asset Storage)│    │  (Monitoring)   │
└─────────────────┘    └─────────────────┘
```

This architecture provides the foundation for building a high-performance, scalable game creation platform that can handle thousands of concurrent users creating and playing games in real-time.