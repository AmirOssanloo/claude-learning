# Asset Pipeline Architecture & Guidelines

This document outlines the asset processing pipeline, optimization strategies, and management workflows for ArcadeForge.

## Overview

The asset pipeline is responsible for uploading, processing, optimizing, and delivering game assets (images, audio, video) to ensure optimal performance across all devices. It handles everything from initial user upload to CDN delivery with automatic optimization.

## Pipeline Architecture

### Asset Flow Overview

```
User Upload → Validation → Processing → Optimization → Storage → CDN → Game Runtime
     ↓            ↓           ↓            ↓           ↓        ↓         ↓
File Upload   Security    Format       Compression   S3     CloudFront  Delivery
Validation    Scanning    Conversion   & Resizing   Storage  Distribution
```

### Core Pipeline Components

```typescript
// Main asset pipeline orchestrator
class AssetPipeline {
  private uploadService: UploadService;
  private processingQueue: ProcessingQueue;
  private optimizationService: OptimizationService;
  private storageService: StorageService;
  private cdnService: CDNService;
  
  async processAsset(uploadedFile: UploadedFile, userId: string): Promise<ProcessedAsset> {
    // 1. Initial validation
    const validatedFile = await this.uploadService.validateFile(uploadedFile);
    
    // 2. Security scanning
    await this.uploadService.scanForMalware(validatedFile);
    
    // 3. Queue for processing
    const processingJob = await this.processingQueue.enqueue({
      fileId: validatedFile.id,
      userId,
      priority: this.determinePriority(validatedFile)
    });
    
    // 4. Process and optimize
    const processedAsset = await this.processFile(validatedFile);
    
    // 5. Store in S3 with versioning
    const storedAsset = await this.storageService.store(processedAsset);
    
    // 6. Distribute via CDN
    await this.cdnService.invalidateCache(storedAsset.urls);
    
    // 7. Update database
    await this.updateAssetDatabase(storedAsset);
    
    return storedAsset;
  }
  
  private async processFile(file: ValidatedFile): Promise<ProcessedAsset> {
    switch (file.type) {
      case 'image':
        return this.processImage(file);
      case 'audio':
        return this.processAudio(file);
      case 'video':
        return this.processVideo(file);
      default:
        throw new Error(`Unsupported file type: ${file.type}`);
    }
  }
}
```

## Image Processing Pipeline

### Multi-Resolution Generation

```typescript
class ImageProcessor {
  private sharp = require('sharp');
  
  async processImage(imageFile: ImageFile): Promise<ProcessedImage> {
    const originalImage = this.sharp(imageFile.buffer);
    const metadata = await originalImage.metadata();
    
    // Generate multiple resolutions for different use cases
    const variants = await Promise.all([
      this.generateThumbnail(originalImage),      // 128x128 for UI
      this.generatePreview(originalImage),        // 512x512 for editor
      this.generateWebOptimized(originalImage),   // Optimized for web
      this.generateRetina(originalImage),         // 2x resolution for high-DPI
      this.generateSprite(originalImage)          // Game-ready sprite
    ]);
    
    // Generate WebP versions for modern browsers
    const webpVariants = await this.generateWebPVersions(variants);
    
    // Create sprite sheets if multiple frames detected
    const spriteSheet = await this.generateSpriteSheet(imageFile);
    
    return {
      id: imageFile.id,
      original: {
        url: await this.uploadVariant(imageFile.buffer, 'original'),
        width: metadata.width!,
        height: metadata.height!,
        format: metadata.format!,
        size: imageFile.size
      },
      variants: variants.concat(webpVariants),
      spriteSheet,
      metadata: {
        hasTransparency: metadata.hasAlpha,
        colorSpace: metadata.space,
        density: metadata.density,
        channels: metadata.channels
      }
    };
  }
  
  private async generateWebOptimized(image: Sharp): Promise<ImageVariant> {
    const buffer = await image
      .jpeg({ quality: 85, progressive: true })
      .toBuffer();
    
    return {
      type: 'web-optimized',
      format: 'jpeg',
      buffer,
      url: await this.uploadVariant(buffer, 'web-optimized'),
      size: buffer.length,
      quality: 85
    };
  }
  
  private async generateSprite(image: Sharp): Promise<ImageVariant> {
    const metadata = await image.metadata();
    
    // Optimize for game sprites
    let optimized: Sharp;
    
    if (metadata.hasAlpha) {
      // PNG for transparency
      optimized = image.png({ 
        quality: 90,
        compressionLevel: 9,
        palette: true // Use palette if suitable
      });
    } else {
      // JPEG for opaque images
      optimized = image.jpeg({ 
        quality: 90,
        progressive: false // Faster decoding for games
      });
    }
    
    const buffer = await optimized.toBuffer();
    
    return {
      type: 'sprite',
      format: metadata.hasAlpha ? 'png' : 'jpeg',
      buffer,
      url: await this.uploadVariant(buffer, 'sprite'),
      size: buffer.length,
      optimizedForGames: true
    };
  }
  
  // Advanced sprite sheet generation
  private async generateSpriteSheet(imageFile: ImageFile): Promise<SpriteSheet | null> {
    // Detect if image contains multiple frames (animation)
    const frames = await this.detectAnimationFrames(imageFile);
    
    if (frames.length <= 1) return null;
    
    // Calculate optimal atlas layout
    const layout = this.calculateAtlasLayout(frames);
    
    // Create sprite sheet
    const atlas = this.sharp({
      create: {
        width: layout.width,
        height: layout.height,
        channels: 4,
        background: { r: 0, g: 0, b: 0, alpha: 0 }
      }
    });
    
    const composite = frames.map((frame, index) => ({
      input: frame.buffer,
      left: layout.positions[index].x,
      top: layout.positions[index].y
    }));
    
    const spriteSheetBuffer = await atlas.composite(composite).png().toBuffer();
    
    return {
      texture: {
        url: await this.uploadVariant(spriteSheetBuffer, 'spritesheet'),
        width: layout.width,
        height: layout.height
      },
      frames: frames.map((frame, index) => ({
        id: `frame_${index}`,
        x: layout.positions[index].x,
        y: layout.positions[index].y,
        width: frame.width,
        height: frame.height,
        duration: frame.duration || 100
      }))
    };
  }
}
```

### Image Optimization Strategies

```typescript
class ImageOptimizer {
  async optimizeForWeb(image: Sharp): Promise<OptimizedImage[]> {
    const metadata = await image.metadata();
    const results: OptimizedImage[] = [];
    
    // Try different formats and pick the best
    const formats = ['webp', 'avif', 'jpeg', 'png'];
    
    for (const format of formats) {
      try {
        const optimized = await this.optimizeFormat(image, format, metadata);
        results.push(optimized);
      } catch (error) {
        console.warn(`Failed to optimize as ${format}:`, error);
      }
    }
    
    // Sort by file size (smallest first)
    return results.sort((a, b) => a.size - b.size);
  }
  
  private async optimizeFormat(image: Sharp, format: string, metadata: Metadata): Promise<OptimizedImage> {
    let processor: Sharp;
    
    switch (format) {
      case 'webp':
        processor = image.webp({ 
          quality: 85,
          effort: 6,
          lossless: metadata.hasAlpha && metadata.width! * metadata.height! < 100000
        });
        break;
        
      case 'avif':
        processor = image.avif({ 
          quality: 80,
          effort: 9
        });
        break;
        
      case 'jpeg':
        if (metadata.hasAlpha) {
          throw new Error('JPEG does not support transparency');
        }
        processor = image.jpeg({ 
          quality: 85,
          progressive: true,
          mozjpeg: true
        });
        break;
        
      case 'png':
        processor = image.png({ 
          quality: 90,
          compressionLevel: 9,
          palette: this.shouldUsePalette(metadata)
        });
        break;
        
      default:
        throw new Error(`Unsupported format: ${format}`);
    }
    
    const buffer = await processor.toBuffer();
    
    return {
      format,
      buffer,
      size: buffer.length,
      compressionRatio: buffer.length / (metadata.width! * metadata.height! * 4),
      url: await this.uploadOptimized(buffer, format)
    };
  }
  
  private shouldUsePalette(metadata: Metadata): boolean {
    // Use palette for small images with limited colors
    return metadata.width! * metadata.height! < 65536 && metadata.channels! <= 3;
  }
}
```

## Audio Processing Pipeline

```typescript
class AudioProcessor {
  private ffmpeg = require('fluent-ffmpeg');
  
  async processAudio(audioFile: AudioFile): Promise<ProcessedAudio> {
    const metadata = await this.getAudioMetadata(audioFile);
    
    // Generate multiple formats and quality levels
    const variants = await Promise.all([
      this.generateWebOptimized(audioFile),    // OGG Vorbis for web
      this.generateMobileOptimized(audioFile), // AAC for mobile
      this.generateHighQuality(audioFile),     // Lossless for music
      this.generateCompressed(audioFile)       // Highly compressed for SFX
    ]);
    
    // Generate waveform visualization
    const waveform = await this.generateWaveform(audioFile);
    
    return {
      id: audioFile.id,
      original: {
        url: await this.uploadVariant(audioFile.buffer, 'original'),
        format: metadata.format,
        duration: metadata.duration,
        sampleRate: metadata.sampleRate,
        bitRate: metadata.bitRate,
        size: audioFile.size
      },
      variants,
      waveform,
      metadata: {
        channels: metadata.channels,
        hasLoudnessNormalization: true,
        peakLevel: metadata.peakLevel,
        rmsLevel: metadata.rmsLevel
      }
    };
  }
  
  private async generateWebOptimized(audioFile: AudioFile): Promise<AudioVariant> {
    return new Promise((resolve, reject) => {
      const outputPath = `/tmp/${audioFile.id}_web.ogg`;
      
      this.ffmpeg(audioFile.buffer)
        .audioCodec('libvorbis')
        .audioBitrate('96k')
        .audioChannels(2)
        .audioFrequency(44100)
        .format('ogg')
        // Normalize audio levels
        .audioFilters([
          'loudnorm=I=-16:TP=-1.5:LRA=11',
          'highpass=f=80',  // Remove sub-bass rumble
          'lowpass=f=15000' // Remove unnecessary high frequencies
        ])
        .save(outputPath)
        .on('end', async () => {
          const buffer = fs.readFileSync(outputPath);
          resolve({
            type: 'web-optimized',
            format: 'ogg',
            codec: 'vorbis',
            bitRate: 96,
            buffer,
            url: await this.uploadVariant(buffer, 'web-optimized'),
            size: buffer.length
          });
          fs.unlinkSync(outputPath);
        })
        .on('error', reject);
    });
  }
  
  private async generateWaveform(audioFile: AudioFile): Promise<Waveform> {
    return new Promise((resolve, reject) => {
      const outputPath = `/tmp/${audioFile.id}_waveform.json`;
      
      this.ffmpeg(audioFile.buffer)
        .audioFilters([
          'showwavespic=s=800x200:colors=0x3498db',
          'scale=800:200'
        ])
        .format('png')
        .save(outputPath.replace('.json', '.png'))
        .on('end', async () => {
          // Generate peak data for interactive waveform
          const peaks = await this.extractPeakData(audioFile, 800);
          
          const waveformImage = fs.readFileSync(outputPath.replace('.json', '.png'));
          
          resolve({
            imageUrl: await this.uploadVariant(waveformImage, 'waveform'),
            peaks,
            width: 800,
            height: 200
          });
          
          fs.unlinkSync(outputPath.replace('.json', '.png'));
        })
        .on('error', reject);
    });
  }
  
  private async extractPeakData(audioFile: AudioFile, samples: number): Promise<number[]> {
    // Extract audio peak data for waveform visualization
    return new Promise((resolve, reject) => {
      const peaks: number[] = [];
      
      this.ffmpeg(audioFile.buffer)
        .audioFilters([
          `aresample=${samples}`,
          'astats=metadata=1:reset=1'
        ])
        .format('null')
        .on('progress', (progress) => {
          // Extract peak values from progress data
          if (progress.frames) {
            peaks.push(progress.currentKbps || 0);
          }
        })
        .on('end', () => resolve(peaks))
        .on('error', reject)
        .run();
    });
  }
}
```

## Storage & CDN Management

```typescript
class StorageService {
  private s3Client: AWS.S3;
  private bucketName: string;
  private cdnDomain: string;
  
  constructor() {
    this.s3Client = new AWS.S3({
      region: process.env.AWS_REGION,
      accessKeyId: process.env.AWS_ACCESS_KEY_ID,
      secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
    });
    this.bucketName = process.env.S3_BUCKET_NAME!;
    this.cdnDomain = process.env.CDN_DOMAIN!;
  }
  
  async storeAsset(asset: ProcessedAsset): Promise<StoredAsset> {
    const assetKey = this.generateAssetKey(asset);
    
    // Upload original and all variants
    const uploadPromises = [
      this.uploadToS3(asset.original.buffer, `${assetKey}/original.${asset.original.format}`),
      ...asset.variants.map(variant => 
        this.uploadToS3(variant.buffer, `${assetKey}/${variant.type}.${variant.format}`)
      )
    ];
    
    await Promise.all(uploadPromises);
    
    // Generate CDN URLs
    const urls = {
      original: `${this.cdnDomain}/${assetKey}/original.${asset.original.format}`,
      variants: asset.variants.reduce((acc, variant) => {
        acc[variant.type] = `${this.cdnDomain}/${assetKey}/${variant.type}.${variant.format}`;
        return acc;
      }, {} as Record<string, string>)
    };
    
    return {
      ...asset,
      s3Key: assetKey,
      urls,
      uploadedAt: new Date()
    };
  }
  
  private async uploadToS3(buffer: Buffer, key: string): Promise<void> {
    const params: AWS.S3.PutObjectRequest = {
      Bucket: this.bucketName,
      Key: key,
      Body: buffer,
      ContentType: this.getContentType(key),
      CacheControl: 'public, max-age=31536000', // 1 year cache
      Metadata: {
        uploadedAt: new Date().toISOString(),
        processedBy: 'arcadeforge-pipeline'
      }
    };
    
    await this.s3Client.upload(params).promise();
  }
  
  // Intelligent asset versioning
  async versionAsset(originalAssetId: string, newAsset: ProcessedAsset): Promise<StoredAsset> {
    const originalAsset = await this.getAsset(originalAssetId);
    
    // Check if significant changes warrant new version
    const needsNewVersion = await this.shouldCreateNewVersion(originalAsset, newAsset);
    
    if (needsNewVersion) {
      // Store as new version
      const versionedAsset = await this.storeAsset(newAsset);
      
      // Update version chain
      await this.updateVersionChain(originalAssetId, versionedAsset.id);
      
      return versionedAsset;
    } else {
      // Replace existing asset
      return this.replaceAsset(originalAssetId, newAsset);
    }
  }
  
  private async shouldCreateNewVersion(original: StoredAsset, updated: ProcessedAsset): boolean {
    // Create new version if:
    // 1. Dimensions changed significantly
    // 2. File size changed by more than 20%
    // 3. Format changed
    // 4. Asset type changed
    
    const dimensionChange = Math.abs(
      (original.original.width - updated.original.width) / original.original.width
    );
    
    const sizeChange = Math.abs(
      (original.original.size - updated.original.size) / original.original.size
    );
    
    return (
      dimensionChange > 0.1 ||
      sizeChange > 0.2 ||
      original.original.format !== updated.original.format
    );
  }
}
```

## Asset Delivery Optimization

```typescript
class AssetDeliveryService {
  private cdnService: CDNService;
  private analyticsService: AnalyticsService;
  
  // Smart asset selection based on device and network
  selectOptimalAsset(assetId: string, context: DeliveryContext): string {
    const asset = this.getAsset(assetId);
    const variants = asset.variants;
    
    // Device capabilities
    const supportsWebP = context.browserSupports.webp;
    const supportsAVIF = context.browserSupports.avif;
    const isRetina = context.devicePixelRatio >= 2;
    const isSlowNetwork = context.connectionType === '2g' || context.connectionType === 'slow-3g';
    
    // Select best format
    let selectedVariant: AssetVariant;
    
    if (isSlowNetwork) {
      // Prioritize smallest file size
      selectedVariant = variants
        .filter(v => v.type === 'compressed')
        .sort((a, b) => a.size - b.size)[0];
    } else if (supportsAVIF) {
      // Best modern format
      selectedVariant = variants.find(v => v.format === 'avif') || 
                        variants.find(v => v.format === 'webp') ||
                        variants.find(v => v.type === 'web-optimized');
    } else if (supportsWebP) {
      selectedVariant = variants.find(v => v.format === 'webp') ||
                        variants.find(v => v.type === 'web-optimized');
    } else {
      selectedVariant = variants.find(v => v.type === 'web-optimized');
    }
    
    // Add resolution selection for retina displays
    if (isRetina && !isSlowNetwork) {
      const retinaVariant = variants.find(v => v.type === 'retina');
      if (retinaVariant && retinaVariant.size < selectedVariant!.size * 1.5) {
        selectedVariant = retinaVariant;
      }
    }
    
    // Track delivery analytics
    this.analyticsService.recordAssetDelivery({
      assetId,
      variantSelected: selectedVariant!.type,
      context
    });
    
    return selectedVariant!.url;
  }
  
  // Progressive loading strategy
  generateProgressiveUrls(assetId: string): ProgressiveLoadingUrls {
    const asset = this.getAsset(assetId);
    
    return {
      placeholder: asset.variants.find(v => v.type === 'thumbnail')?.url,
      preview: asset.variants.find(v => v.type === 'preview')?.url,
      full: asset.variants.find(v => v.type === 'web-optimized')?.url,
      retina: asset.variants.find(v => v.type === 'retina')?.url
    };
  }
}
```

## Quality Assurance & Validation

```typescript
class AssetValidator {
  // Comprehensive asset validation
  async validateAsset(asset: ProcessedAsset): Promise<ValidationResult> {
    const results: ValidationResult = {
      valid: true,
      warnings: [],
      errors: [],
      recommendations: []
    };
    
    // File size validation
    if (asset.original.size > 10 * 1024 * 1024) { // 10MB
      results.warnings.push('Asset is larger than 10MB, consider optimization');
    }
    
    // Resolution validation
    if (asset.original.width > 4096 || asset.original.height > 4096) {
      results.recommendations.push('Consider reducing resolution for better performance');
    }
    
    // Format validation
    if (asset.type === 'image') {
      await this.validateImage(asset as ProcessedImage, results);
    } else if (asset.type === 'audio') {
      await this.validateAudio(asset as ProcessedAudio, results);
    }
    
    // Performance impact assessment
    const performanceScore = await this.assessPerformanceImpact(asset);
    if (performanceScore < 0.7) {
      results.warnings.push(`Performance score: ${performanceScore.toFixed(2)} - consider optimization`);
    }
    
    // Accessibility checks
    await this.validateAccessibility(asset, results);
    
    results.valid = results.errors.length === 0;
    return results;
  }
  
  private async validateImage(image: ProcessedImage, results: ValidationResult): Promise<void> {
    // Check for appropriate formats
    if (image.metadata.hasTransparency && !image.variants.some(v => v.format === 'png' || v.format === 'webp')) {
      results.errors.push('Transparent image should have PNG or WebP variant');
    }
    
    // Validate sprite sheets
    if (image.spriteSheet) {
      const totalFrames = image.spriteSheet.frames.length;
      if (totalFrames > 100) {
        results.warnings.push(`Sprite sheet has ${totalFrames} frames, consider splitting`);
      }
      
      // Check frame consistency
      const frameSizes = image.spriteSheet.frames.map(f => f.width * f.height);
      const inconsistentSizes = new Set(frameSizes).size > 1;
      if (inconsistentSizes) {
        results.recommendations.push('Sprite frames have inconsistent sizes');
      }
    }
    
    // Color depth analysis
    if (image.metadata.channels > 3 && !image.metadata.hasTransparency) {
      results.recommendations.push('Image has alpha channel but no transparency, consider removing');
    }
  }
  
  private async validateAudio(audio: ProcessedAudio, results: ValidationResult): Promise<void> {
    // Duration validation
    if (audio.original.duration > 300) { // 5 minutes
      results.warnings.push('Audio file is longer than 5 minutes, consider streaming');
    }
    
    // Sample rate validation
    if (audio.original.sampleRate > 48000) {
      results.recommendations.push('Sample rate above 48kHz may be unnecessary for games');
    }
    
    // Loudness validation
    if (audio.metadata.peakLevel > -3) {
      results.warnings.push('Audio peaks above -3dB, may cause distortion');
    }
    
    if (audio.metadata.rmsLevel < -30) {
      results.warnings.push('Audio RMS level very low, may be difficult to hear');
    }
  }
  
  private async assessPerformanceImpact(asset: ProcessedAsset): Promise<number> {
    let score = 1.0;
    
    // File size impact (0-1 scale)
    const sizeScore = Math.max(0, 1 - (asset.original.size / (5 * 1024 * 1024))); // 5MB baseline
    score *= sizeScore;
    
    // Resolution impact for images
    if (asset.type === 'image') {
      const pixelCount = asset.original.width * asset.original.height;
      const resolutionScore = Math.max(0, 1 - (pixelCount / (2048 * 2048))); // 2K baseline
      score *= resolutionScore;
    }
    
    // Format efficiency
    const hasModernFormats = asset.variants.some(v => v.format === 'webp' || v.format === 'avif');
    if (!hasModernFormats) {
      score *= 0.8;
    }
    
    return score;
  }
}
```

## Monitoring & Analytics

```typescript
class AssetAnalyticsService {
  private metricsCollector: MetricsCollector;
  
  recordAssetUsage(assetId: string, context: UsageContext): void {
    this.metricsCollector.increment('asset.usage', {
      assetId,
      gameId: context.gameId,
      userId: context.userId,
      platform: context.platform
    });
  }
  
  recordLoadingPerformance(assetId: string, loadTime: number, size: number): void {
    this.metricsCollector.histogram('asset.loading.time', loadTime, {
      assetId,
      sizeCategory: this.getSizeCategory(size)
    });
    
    this.metricsCollector.histogram('asset.loading.throughput', size / loadTime, {
      assetId
    });
  }
  
  async generateOptimizationReport(timeRange: TimeRange): Promise<OptimizationReport> {
    const slowestAssets = await this.getSlowLoadingAssets(timeRange);
    const largestAssets = await this.getLargestUnoptimizedAssets(timeRange);
    const unusedAssets = await this.getUnusedAssets(timeRange);
    
    return {
      slowestAssets: slowestAssets.map(asset => ({
        ...asset,
        recommendation: this.generateOptimizationRecommendation(asset)
      })),
      largestAssets,
      unusedAssets,
      totalSavingsPotential: this.calculateSavingsPotential(largestAssets)
    };
  }
  
  private generateOptimizationRecommendation(asset: AssetMetrics): string {
    if (asset.averageLoadTime > 2000) {
      return 'Consider reducing file size or using progressive loading';
    }
    
    if (asset.size > 1024 * 1024 && !asset.hasModernFormats) {
      return 'Convert to WebP or AVIF format for better compression';
    }
    
    if (asset.type === 'image' && asset.dimensions.width > 2048) {
      return 'Consider reducing resolution for web use';
    }
    
    return 'Asset is well optimized';
  }
}
```

## Development Guidelines

### Asset Standards
- **Image Formats**: WebP/AVIF preferred, PNG for transparency, JPEG for photos
- **Audio Formats**: OGG Vorbis for web, AAC for mobile compatibility
- **Resolution Limits**: Max 4K for images, 48kHz for audio
- **File Size Limits**: 10MB per asset, 50MB total per game project

### Performance Requirements
- **Loading Time**: <2s for individual assets over 3G
- **Processing Time**: <30s for asset pipeline processing
- **Storage Efficiency**: >50% compression ratio for optimized variants
- **CDN Cache Hit Rate**: >95% for frequently accessed assets

### Quality Assurance
- **Automated Validation**: All uploaded assets must pass validation
- **Security Scanning**: Malware detection for all file uploads
- **Format Verification**: Ensure file contents match declared MIME type
- **Accessibility**: Alternative text and descriptions for media assets

### Monitoring Requirements
- **Usage Tracking**: Monitor asset usage patterns and performance
- **Error Logging**: Track processing failures and optimization issues
- **Performance Metrics**: Load times, bandwidth usage, cache efficiency
- **Cost Optimization**: Monitor storage and CDN costs per asset

This asset pipeline architecture ensures optimal performance, scalability, and user experience while maintaining high quality standards across all game assets.