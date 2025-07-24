# Game Engine Development Guidelines

This document outlines the architecture and development guidelines for the ArcadeForge 2D game runtime engine.

## Overview

The ArcadeForge game engine is a lightweight, high-performance 2D game runtime built on Pixi.js. It's designed to run games created in the visual editor with consistent 60fps performance across desktop, mobile, and tablet devices.

## Engine Architecture

### Core Systems

```typescript
// Main engine architecture components
class GameEngine {
  private pixiApp: PIXI.Application;
  private sceneManager: SceneManager;
  private inputSystem: InputSystem;
  private physicsSystem: PhysicsSystem;
  private audioSystem: AudioSystem;
  private assetLoader: AssetLoader;
  
  constructor(config: GameConfig) {
    this.initializeRenderer(config);
    this.initializeSystems();
    this.startGameLoop();
  }
  
  private gameLoop = (deltaTime: number) => {
    // Fixed timestep physics simulation
    this.physicsSystem.update(deltaTime);
    
    // Update game logic
    this.sceneManager.update(deltaTime);
    
    // Process input
    this.inputSystem.update();
    
    // Render frame
    this.sceneManager.render();
    
    // Performance monitoring
    this.monitorPerformance();
  };
}
```

### Scene Management System

```typescript
interface Scene {
  id: string;
  name: string;
  objects: GameObject[];
  camera: Camera;
  physics: PhysicsWorld;
  bounds: Rectangle;
}

class SceneManager {
  private activeScene: Scene | null = null;
  private sceneCache = new Map<string, Scene>();
  private transitionState: TransitionState | null = null;
  
  async loadScene(sceneData: SceneData): Promise<void> {
    // Preload scene assets
    await this.assetLoader.loadSceneAssets(sceneData);
    
    // Build scene from data
    const scene = await this.buildScene(sceneData);
    
    // Cache for fast switching
    this.sceneCache.set(scene.id, scene);
    
    // Transition to new scene
    this.transitionToScene(scene);
  }
  
  private async buildScene(sceneData: SceneData): Promise<Scene> {
    const scene: Scene = {
      id: sceneData.id,
      name: sceneData.name,
      objects: [],
      camera: new Camera(sceneData.camera),
      physics: new PhysicsWorld(sceneData.physics),
      bounds: sceneData.bounds
    };
    
    // Create game objects from scene data
    for (const objectData of sceneData.objects) {
      const gameObject = await this.createGameObject(objectData);
      scene.objects.push(gameObject);
      scene.physics.addBody(gameObject.physicsBody);
    }
    
    return scene;
  }
  
  update(deltaTime: number): void {
    if (!this.activeScene) return;
    
    // Update all game objects
    for (const object of this.activeScene.objects) {
      object.update(deltaTime);
    }
    
    // Update camera
    this.activeScene.camera.update(deltaTime);
  }
}
```

### Game Object System

```typescript
// Entity-Component-System architecture
interface Component {
  type: string;
  update(deltaTime: number): void;
  destroy(): void;
}

class GameObject {
  id: string;
  transform: Transform;
  components = new Map<string, Component>();
  active = true;
  
  constructor(id: string, transform: Transform) {
    this.id = id;
    this.transform = transform;
  }
  
  addComponent<T extends Component>(component: T): T {
    this.components.set(component.type, component);
    return component;
  }
  
  getComponent<T extends Component>(type: string): T | null {
    return this.components.get(type) as T || null;
  }
  
  update(deltaTime: number): void {
    if (!this.active) return;
    
    for (const component of this.components.values()) {
      component.update(deltaTime);
    }
  }
  
  destroy(): void {
    for (const component of this.components.values()) {
      component.destroy();
    }
    this.components.clear();
  }
}

// Essential components
class SpriteComponent implements Component {
  type = 'sprite';
  sprite: PIXI.Sprite;
  
  constructor(texture: PIXI.Texture) {
    this.sprite = new PIXI.Sprite(texture);
  }
  
  update(deltaTime: number): void {
    // Update sprite position from transform
    const transform = this.gameObject.transform;
    this.sprite.position.set(transform.x, transform.y);
    this.sprite.rotation = transform.rotation;
    this.sprite.scale.set(transform.scaleX, transform.scaleY);
  }
  
  destroy(): void {
    this.sprite.destroy();
  }
}

class PhysicsComponent implements Component {
  type = 'physics';
  body: PhysicsBody;
  
  constructor(body: PhysicsBody) {
    this.body = body;
  }
  
  update(deltaTime: number): void {
    // Sync transform with physics body
    const transform = this.gameObject.transform;
    transform.x = this.body.position.x;
    transform.y = this.body.position.y;
    transform.rotation = this.body.rotation;
  }
  
  destroy(): void {
    this.body.destroy();
  }
}
```

## Physics System

### Collision Detection

```typescript
class PhysicsSystem {
  private bodies: PhysicsBody[] = [];
  private spatialHash: SpatialHash;
  private gravity = { x: 0, y: 980 }; // Pixels per second squared
  
  constructor() {
    this.spatialHash = new SpatialHash(64); // 64x64 pixel cells
  }
  
  update(deltaTime: number): void {
    const dt = Math.min(deltaTime, 1/30); // Cap delta time for stability
    
    // Apply forces (gravity, input, etc.)
    this.applyForces(dt);
    
    // Integrate velocity
    this.integrateVelocity(dt);
    
    // Detect and resolve collisions
    this.detectCollisions();
    this.resolveCollisions();
    
    // Update spatial hash
    this.updateSpatialHash();
  }
  
  private detectCollisions(): void {
    const collisions: Collision[] = [];
    
    // Use spatial hash for broad phase collision detection
    const potentialPairs = this.spatialHash.getPotentialCollisions();
    
    for (const [bodyA, bodyB] of potentialPairs) {
      if (this.narrowPhaseCollision(bodyA, bodyB)) {
        collisions.push(new Collision(bodyA, bodyB));
      }
    }
    
    this.currentCollisions = collisions;
  }
  
  private narrowPhaseCollision(bodyA: PhysicsBody, bodyB: PhysicsBody): boolean {
    // AABB collision detection
    return (
      bodyA.bounds.x < bodyB.bounds.x + bodyB.bounds.width &&
      bodyA.bounds.x + bodyA.bounds.width > bodyB.bounds.x &&
      bodyA.bounds.y < bodyB.bounds.y + bodyB.bounds.height &&
      bodyA.bounds.y + bodyA.bounds.height > bodyB.bounds.y
    );
  }
}

// Spatial hash for efficient collision detection
class SpatialHash {
  private cellSize: number;
  private cells = new Map<string, PhysicsBody[]>();
  
  constructor(cellSize: number) {
    this.cellSize = cellSize;
  }
  
  insert(body: PhysicsBody): void {
    const cells = this.getCellsForBody(body);
    
    for (const cellKey of cells) {
      if (!this.cells.has(cellKey)) {
        this.cells.set(cellKey, []);
      }
      this.cells.get(cellKey)!.push(body);
    }
  }
  
  private getCellsForBody(body: PhysicsBody): string[] {
    const bounds = body.bounds;
    const cells: string[] = [];
    
    const startX = Math.floor(bounds.x / this.cellSize);
    const endX = Math.floor((bounds.x + bounds.width) / this.cellSize);
    const startY = Math.floor(bounds.y / this.cellSize);
    const endY = Math.floor((bounds.y + bounds.height) / this.cellSize);
    
    for (let x = startX; x <= endX; x++) {
      for (let y = startY; y <= endY; y++) {
        cells.push(`${x},${y}`);
      }
    }
    
    return cells;
  }
}
```

### Platformer Physics

```typescript
class PlatformerController implements Component {
  type = 'platformer';
  
  private velocity = { x: 0, y: 0 };
  private onGround = false;
  private jumpForce = 400;
  private moveSpeed = 200;
  private jumpBufferTime = 0.1; // seconds
  private coyoteTime = 0.1; // seconds
  
  update(deltaTime: number): void {
    const input = this.inputSystem.getInput();
    
    // Horizontal movement
    if (input.left) {
      this.velocity.x = -this.moveSpeed;
    } else if (input.right) {
      this.velocity.x = this.moveSpeed;
    } else {
      // Apply friction when not moving
      this.velocity.x *= 0.8;
    }
    
    // Jumping with buffer and coyote time
    if (input.jump && this.canJump()) {
      this.velocity.y = -this.jumpForce;
      this.onGround = false;
    }
    
    // Apply gravity
    if (!this.onGround) {
      this.velocity.y += 980 * deltaTime; // gravity
    }
    
    // Update position
    const body = this.gameObject.getComponent<PhysicsComponent>('physics').body;
    body.velocity = this.velocity;
  }
  
  private canJump(): boolean {
    return this.onGround || this.getCoyoteTimeRemaining() > 0;
  }
  
  onCollision(collision: Collision): void {
    if (collision.normal.y < -0.5) {
      // Landing on ground
      this.onGround = true;
      this.velocity.y = 0;
    }
    
    if (collision.normal.x !== 0) {
      // Hit wall
      this.velocity.x = 0;
    }
  }
}
```

## Audio System

```typescript
class AudioSystem {
  private sounds = new Map<string, Howl>();
  private musicStack: Howl[] = [];
  private masterVolume = 1.0;
  private musicVolume = 0.7;
  private sfxVolume = 1.0;
  
  loadSound(id: string, url: string): Promise<void> {
    return new Promise((resolve, reject) => {
      const sound = new Howl({
        src: [url],
        volume: this.sfxVolume * this.masterVolume,
        onload: () => resolve(),
        onloaderror: () => reject(new Error(`Failed to load sound: ${url}`))
      });
      
      this.sounds.set(id, sound);
    });
  }
  
  playSound(id: string, options?: { volume?: number; loop?: boolean }): number {
    const sound = this.sounds.get(id);
    if (!sound) {
      console.warn(`Sound not found: ${id}`);
      return -1;
    }
    
    const volume = (options?.volume ?? 1) * this.sfxVolume * this.masterVolume;
    sound.volume(volume);
    sound.loop(options?.loop ?? false);
    
    return sound.play();
  }
  
  playMusic(id: string, fadeInTime = 1000): void {
    const music = this.sounds.get(id);
    if (!music) return;
    
    // Fade out current music
    if (this.musicStack.length > 0) {
      const currentMusic = this.musicStack[this.musicStack.length - 1];
      currentMusic.fade(currentMusic.volume(), 0, fadeInTime);
      currentMusic.once('fade', () => currentMusic.stop());
    }
    
    // Fade in new music
    music.volume(0);
    music.loop(true);
    const playId = music.play();
    music.fade(0, this.musicVolume * this.masterVolume, fadeInTime);
    
    this.musicStack.push(music);
  }
  
  // Spatial audio for positioned sounds
  playSpatialSound(id: string, position: { x: number; y: number }, listenerPosition: { x: number; y: number }): void {
    const sound = this.sounds.get(id);
    if (!sound) return;
    
    const distance = Math.sqrt(
      Math.pow(position.x - listenerPosition.x, 2) +
      Math.pow(position.y - listenerPosition.y, 2)
    );
    
    // Calculate volume based on distance (adjust falloff as needed)
    const maxDistance = 500;
    const volume = Math.max(0, 1 - (distance / maxDistance));
    
    // Calculate stereo panning
    const panRange = 200;
    const pan = Math.max(-1, Math.min(1, (position.x - listenerPosition.x) / panRange));
    
    const playId = sound.play();
    sound.volume(volume * this.sfxVolume * this.masterVolume, playId);
    sound.stereo(pan, playId);
  }
}
```

## Performance Optimization

### Object Pooling

```typescript
class ObjectPool<T> {
  private pool: T[] = [];
  private factory: () => T;
  private reset: (obj: T) => void;
  private maxSize: number;
  
  constructor(factory: () => T, reset: (obj: T) => void, maxSize = 100) {
    this.factory = factory;
    this.reset = reset;
    this.maxSize = maxSize;
  }
  
  acquire(): T {
    if (this.pool.length > 0) {
      return this.pool.pop()!;
    }
    return this.factory();
  }
  
  release(obj: T): void {
    if (this.pool.length < this.maxSize) {
      this.reset(obj);
      this.pool.push(obj);
    }
  }
}

// Usage for particles, bullets, etc.
const bulletPool = new ObjectPool(
  () => new Bullet(),
  (bullet) => {
    bullet.reset();
    bullet.active = false;
  },
  50
);
```

### Frame Rate Monitoring

```typescript
class PerformanceMonitor {
  private frameTimeHistory: number[] = [];
  private targetFrameTime = 16.67; // 60fps
  private performanceWarningThreshold = 20; // 50fps
  
  beginFrame(): void {
    this.frameStartTime = performance.now();
  }
  
  endFrame(): void {
    const frameTime = performance.now() - this.frameStartTime;
    this.frameTimeHistory.push(frameTime);
    
    // Keep last 60 frames
    if (this.frameTimeHistory.length > 60) {
      this.frameTimeHistory.shift();
    }
    
    // Check for performance issues
    if (frameTime > this.performanceWarningThreshold) {
      this.handlePerformanceWarning();
    }
  }
  
  private handlePerformanceWarning(): void {
    // Automatically reduce quality settings
    this.reduceParticleCount();
    this.disableExpensiveEffects();
    this.increasePhysicsStepSize();
  }
  
  getAverageFrameTime(): number {
    if (this.frameTimeHistory.length === 0) return 0;
    
    const sum = this.frameTimeHistory.reduce((a, b) => a + b, 0);
    return sum / this.frameTimeHistory.length;
  }
  
  getFPS(): number {
    const avgFrameTime = this.getAverageFrameTime();
    return avgFrameTime > 0 ? 1000 / avgFrameTime : 0;
  }
}
```

## Input System

```typescript
class InputSystem {
  private keys = new Set<string>();
  private mousePosition = { x: 0, y: 0 };
  private mouseButtons = new Set<number>();
  private touchPoints = new Map<number, { x: number; y: number }>();
  
  constructor() {
    this.bindEvents();
  }
  
  private bindEvents(): void {
    // Keyboard events
    window.addEventListener('keydown', (e) => {
      this.keys.add(e.code);
    });
    
    window.addEventListener('keyup', (e) => {
      this.keys.delete(e.code);
    });
    
    // Mouse events
    window.addEventListener('mousemove', (e) => {
      this.mousePosition = { x: e.clientX, y: e.clientY };
    });
    
    window.addEventListener('mousedown', (e) => {
      this.mouseButtons.add(e.button);
    });
    
    window.addEventListener('mouseup', (e) => {
      this.mouseButtons.delete(e.button);
    });
    
    // Touch events
    window.addEventListener('touchstart', (e) => {
      for (const touch of e.changedTouches) {
        this.touchPoints.set(touch.identifier, {
          x: touch.clientX,
          y: touch.clientY
        });
      }
    });
    
    window.addEventListener('touchmove', (e) => {
      for (const touch of e.changedTouches) {
        this.touchPoints.set(touch.identifier, {
          x: touch.clientX,
          y: touch.clientY
        });
      }
    });
    
    window.addEventListener('touchend', (e) => {
      for (const touch of e.changedTouches) {
        this.touchPoints.delete(touch.identifier);
      }
    });
  }
  
  // High-level input queries
  isKeyPressed(keyCode: string): boolean {
    return this.keys.has(keyCode);
  }
  
  getAxis(negative: string, positive: string): number {
    let value = 0;
    if (this.isKeyPressed(negative)) value -= 1;
    if (this.isKeyPressed(positive)) value += 1;
    return value;
  }
  
  // For platformer games
  getPlatformerInput(): PlatformerInput {
    return {
      left: this.isKeyPressed('ArrowLeft') || this.isKeyPressed('KeyA'),
      right: this.isKeyPressed('ArrowRight') || this.isKeyPressed('KeyD'),
      jump: this.isKeyPressed('Space') || this.isKeyPressed('ArrowUp') || this.isKeyPressed('KeyW'),
      action: this.isKeyPressed('KeyE') || this.isKeyPressed('Enter'),
      horizontal: this.getAxis('ArrowLeft', 'ArrowRight') || this.getAxis('KeyA', 'KeyD')
    };
  }
}
```

## Asset Loading & Management

```typescript
class AssetLoader {
  private loadedAssets = new Map<string, any>();
  private loadingPromises = new Map<string, Promise<any>>();
  
  async loadAsset(assetData: AssetData): Promise<any> {
    if (this.loadedAssets.has(assetData.id)) {
      return this.loadedAssets.get(assetData.id);
    }
    
    if (this.loadingPromises.has(assetData.id)) {
      return this.loadingPromises.get(assetData.id);
    }
    
    const loadPromise = this.loadAssetByType(assetData);
    this.loadingPromises.set(assetData.id, loadPromise);
    
    try {
      const asset = await loadPromise;
      this.loadedAssets.set(assetData.id, asset);
      this.loadingPromises.delete(assetData.id);
      return asset;
    } catch (error) {
      this.loadingPromises.delete(assetData.id);
      throw error;
    }
  }
  
  private async loadAssetByType(assetData: AssetData): Promise<any> {
    switch (assetData.type) {
      case 'image':
        return PIXI.Assets.load(assetData.url);
      
      case 'audio':
        return new Promise((resolve, reject) => {
          const audio = new Howl({
            src: [assetData.url],
            onload: () => resolve(audio),
            onloaderror: () => reject(new Error(`Failed to load audio: ${assetData.url}`))
          });
        });
      
      case 'spritesheet':
        const texture = await PIXI.Assets.load(assetData.url);
        return new PIXI.Spritesheet(texture, assetData.metadata);
      
      default:
        throw new Error(`Unsupported asset type: ${assetData.type}`);
    }
  }
  
  // Progressive loading for large scenes
  async loadSceneAssets(sceneData: SceneData, onProgress?: (progress: number) => void): Promise<void> {
    const assetIds = this.extractAssetIds(sceneData);
    const totalAssets = assetIds.length;
    let loadedCount = 0;
    
    // Load assets in parallel with progress tracking
    const loadPromises = assetIds.map(async (assetId) => {
      const assetData = sceneData.assets.find(a => a.id === assetId);
      if (assetData) {
        await this.loadAsset(assetData);
        loadedCount++;
        onProgress?.(loadedCount / totalAssets);
      }
    });
    
    await Promise.all(loadPromises);
  }
}
```

## Error Handling & Recovery

```typescript
class GameEngine {
  private errorRecoveryStrategies = new Map<string, () => void>();
  
  constructor() {
    this.setupErrorHandling();
  }
  
  private setupErrorHandling(): void {
    // Handle uncaught errors
    window.addEventListener('error', (event) => {
      this.handleError(event.error, 'Runtime Error');
    });
    
    // Handle unhandled promise rejections
    window.addEventListener('unhandledrejection', (event) => {
      this.handleError(event.reason, 'Promise Rejection');
    });
    
    // Register recovery strategies
    this.errorRecoveryStrategies.set('AssetLoadError', () => {
      this.useDefaultAssets();
    });
    
    this.errorRecoveryStrategies.set('PhysicsError', () => {
      this.resetPhysicsSystem();
    });
    
    this.errorRecoveryStrategies.set('RenderError', () => {
      this.fallbackToCanvas2D();
    });
  }
  
  private handleError(error: Error, context: string): void {
    console.error(`Game Engine Error (${context}):`, error);
    
    // Try to recover based on error type
    const errorType = this.classifyError(error);
    const recoveryStrategy = this.errorRecoveryStrategies.get(errorType);
    
    if (recoveryStrategy) {
      try {
        recoveryStrategy();
        console.log(`Recovered from ${errorType}`);
      } catch (recoveryError) {
        console.error('Recovery failed:', recoveryError);
        this.criticalErrorShutdown();
      }
    } else {
      this.criticalErrorShutdown();
    }
  }
  
  private criticalErrorShutdown(): void {
    // Save game state to local storage
    this.saveGameState();
    
    // Show error message to user
    this.showErrorMessage();
    
    // Stop game loop
    this.pixiApp.stop();
  }
}
```

## Development Guidelines

### Performance Standards
- **Target Frame Rate**: 60fps consistently
- **Memory Usage**: Monitor and cleanup unused assets
- **Load Times**: Scene loading under 2 seconds
- **Battery Life**: Optimize for mobile device battery consumption

### Code Quality
- **TypeScript Strict**: No `any` types allowed
- **Error Handling**: All async operations must handle errors
- **Testing**: Unit tests for all engine systems
- **Documentation**: JSDoc comments for public APIs

### Platform Compatibility
- **Desktop**: Windows, macOS, Linux browsers
- **Mobile**: iOS Safari, Android Chrome
- **Tablet**: iPad, Android tablets
- **WebGL Support**: Graceful fallback to Canvas 2D

This game engine architecture provides the foundation for high-performance 2D games with consistent behavior across all target platforms.