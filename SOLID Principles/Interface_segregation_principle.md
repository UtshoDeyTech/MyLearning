# Interface Segregation Principle (ISP)

## Table of Contents
1. [Introduction](#introduction)
2. [Definition and Theory](#definition-and-theory)
3. [Why ISP Matters](#why-isp-matters)
4. [When to Apply ISP](#when-to-apply-isp)
5. [Consequences of Violating ISP](#consequences-of-violating-isp)
6. [Python Examples](#python-examples)
7. [Interface Design Patterns](#interface-design-patterns)
8. [Benefits of Following ISP](#benefits-of-following-isp)
9. [Common Pitfalls](#common-pitfalls)
10. [Best Practices](#best-practices)
11. [Conclusion](#conclusion)

## Introduction

The Interface Segregation Principle (ISP) is the fourth principle in the SOLID principles of object-oriented design, introduced by Robert C. Martin. It addresses the problem of "fat" interfaces that force clients to depend on methods they don't use, promoting the creation of focused, cohesive interfaces.

ISP is particularly important in creating maintainable and flexible systems where different clients have different needs from the same service or component.

## Definition and Theory

### Formal Definition
**"Clients should not be forced to depend on interfaces they do not use."**

### Alternative Formulations
- **"Many client-specific interfaces are better than one general-purpose interface"**
- **"No client should be forced to implement methods it doesn't use"**
- **"Keep interfaces small and focused"**

### Core Concepts

#### Interface Pollution
When an interface contains methods that are not relevant to all its clients, it becomes "polluted" with unnecessary dependencies.

#### Role Interfaces
Interfaces should be designed around the roles that clients need to play, not around the capabilities of the implementing classes.

#### Client-Specific Interfaces
Different clients should see different views of the same object through separate, focused interfaces.

#### Fat Interfaces
Large interfaces that combine multiple responsibilities force clients to depend on functionality they don't need.

## Why ISP Matters

### 1. **Reduced Coupling**
Clients only depend on the methods they actually use, reducing unnecessary dependencies.

### 2. **Improved Maintainability**
Changes to unused interface methods don't affect clients that don't use them.

### 3. **Enhanced Testability**
Smaller interfaces are easier to mock and test in isolation.

### 4. **Better Code Organization**
Forces clear separation of concerns and responsibilities.

### 5. **Increased Flexibility**
Clients can implement only the interfaces they need, making the system more flexible.

### 6. **Reduced Compilation Dependencies**
In compiled languages, changes to unused interface methods don't require recompilation of unaffected clients.

## When to Apply ISP

### You Should Apply ISP When:

1. **Designing public APIs or interfaces**
2. **Creating abstract base classes**
3. **Working with dependency injection**
4. **Building plugin architectures**
5. **Designing service interfaces**
6. **Creating contracts between system components**

### Warning Signs That ISP is Violated:

- Interfaces with many unrelated methods
- Classes implementing interfaces but leaving methods empty or throwing "not implemented" exceptions
- Clients importing interfaces just to use one or two methods
- Changes to interface methods affecting unrelated clients
- Classes implementing large interfaces with many unused methods
- Comments like "this method is not applicable for this implementation"
- Mock objects that need to stub many irrelevant methods

## Consequences of Violating ISP

### 1. **Unnecessary Dependencies**
Clients become dependent on methods they never use, creating unnecessary coupling.

### 2. **Difficult Testing**
Large interfaces require complex mocking and stubbing of irrelevant methods.

### 3. **Reduced Flexibility**
Classes are forced to implement methods they don't need, making the system rigid.

### 4. **Maintenance Overhead**
Changes to any part of a fat interface can affect all implementing classes.

### 5. **Violation of Single Responsibility**
Fat interfaces often violate SRP by combining multiple responsibilities.

### 6. **Implementation Burden**
Developers must implement or stub methods they don't need, increasing development overhead.

## Python Examples

### Example 1: Multi-Function Device Interface

#### Violating ISP

```python
# BAD: Violates ISP - Fat interface
from abc import ABC, abstractmethod

class MultiFunctionDevice(ABC):
    """Fat interface that forces all implementations to support all operations"""
    
    @abstractmethod
    def print_document(self, document: str) -> bool:
        pass
    
    @abstractmethod
    def scan_document(self) -> str:
        pass
    
    @abstractmethod
    def fax_document(self, document: str, number: str) -> bool:
        pass
    
    @abstractmethod
    def copy_document(self) -> str:
        pass
    
    @abstractmethod
    def email_document(self, document: str, email: str) -> bool:
        pass
    
    @abstractmethod
    def print_photos(self, photos: list) -> bool:
        pass
    
    @abstractmethod
    def scan_to_cloud(self, cloud_service: str) -> bool:
        pass

class OldPrinter(MultiFunctionDevice):
    """Simple printer forced to implement methods it doesn't support"""
    
    def print_document(self, document: str) -> bool:
        print(f"Printing: {document}")
        return True
    
    def scan_document(self) -> str:
        # Problem: Old printer doesn't have scanner
        raise NotImplementedError("This printer doesn't support scanning")
    
    def fax_document(self, document: str, number: str) -> bool:
        # Problem: Old printer doesn't have fax
        raise NotImplementedError("This printer doesn't support fax")
    
    def copy_document(self) -> str:
        # Problem: Can't copy without scanner
        raise NotImplementedError("This printer doesn't support copying")
    
    def email_document(self, document: str, email: str) -> bool:
        # Problem: Old printer doesn't have network capabilities
        raise NotImplementedError("This printer doesn't support email")
    
    def print_photos(self, photos: list) -> bool:
        # Problem: Old printer can't handle photos
        raise NotImplementedError("This printer doesn't support photo printing")
    
    def scan_to_cloud(self, cloud_service: str) -> bool:
        # Problem: No scanner or network
        raise NotImplementedError("This printer doesn't support cloud scanning")

class SimpleScanner(MultiFunctionDevice):
    """Simple scanner forced to implement printing methods"""
    
    def print_document(self, document: str) -> bool:
        # Problem: Scanner can't print
        raise NotImplementedError("This device is scan-only")
    
    def scan_document(self) -> str:
        print("Scanning document...")
        return "scanned_document_data"
    
    def fax_document(self, document: str, number: str) -> bool:
        # Problem: Scanner doesn't have fax
        raise NotImplementedError("This device doesn't support fax")
    
    def copy_document(self) -> str:
        # Problem: Can scan but can't print the copy
        raise NotImplementedError("This device cannot print copies")
    
    def email_document(self, document: str, email: str) -> bool:
        # Problem: Scanner doesn't have email capabilities
        raise NotImplementedError("This device doesn't support email")
    
    def print_photos(self, photos: list) -> bool:
        # Problem: Scanner can't print
        raise NotImplementedError("This device cannot print")
    
    def scan_to_cloud(self, cloud_service: str) -> bool:
        # Problem: Scanner doesn't have cloud connectivity
        raise NotImplementedError("This device doesn't support cloud services")

class ModernAllInOne(MultiFunctionDevice):
    """Modern device that supports everything"""
    
    def print_document(self, document: str) -> bool:
        print(f"Printing document: {document}")
        return True
    
    def scan_document(self) -> str:
        print("Scanning document...")
        return "scanned_document_data"
    
    def fax_document(self, document: str, number: str) -> bool:
        print(f"Faxing document to {number}")
        return True
    
    def copy_document(self) -> str:
        scanned = self.scan_document()
        self.print_document(scanned)
        return scanned
    
    def email_document(self, document: str, email: str) -> bool:
        print(f"Emailing document to {email}")
        return True
    
    def print_photos(self, photos: list) -> bool:
        print(f"Printing {len(photos)} photos")
        return True
    
    def scan_to_cloud(self, cloud_service: str) -> bool:
        print(f"Scanning to {cloud_service}")
        return True

# Client code that breaks with incomplete implementations
def office_workflow(device: MultiFunctionDevice):
    """This function expects full functionality but breaks with simple devices"""
    try:
        # Try to use all features
        device.print_document("Report")
        scanned = device.scan_document()
        device.copy_document()
        device.fax_document("Contract", "555-1234")
        device.email_document("Invoice", "client@example.com")
        
    except NotImplementedError as e:
        print(f"Workflow failed: {e}")

# Usage - demonstrates ISP violation
devices = [
    OldPrinter(),
    SimpleScanner(), 
    ModernAllInOne()
]

for device in devices:
    print(f"\nTesting {device.__class__.__name__}:")
    office_workflow(device)
```

#### Following ISP

```python
# GOOD: Following ISP - Segregated interfaces
from abc import ABC, abstractmethod
from typing import List

# Core printing interface
class Printer(ABC):
    @abstractmethod
    def print_document(self, document: str) -> bool:
        pass

# Core scanning interface  
class Scanner(ABC):
    @abstractmethod
    def scan_document(self) -> str:
        pass

# Communication interfaces
class FaxMachine(ABC):
    @abstractmethod
    def fax_document(self, document: str, number: str) -> bool:
        pass

class EmailSender(ABC):
    @abstractmethod
    def email_document(self, document: str, email: str) -> bool:
        pass

# Advanced features
class PhotoPrinter(ABC):
    @abstractmethod
    def print_photos(self, photos: List[str]) -> bool:
        pass

class CloudConnector(ABC):
    @abstractmethod
    def scan_to_cloud(self, cloud_service: str) -> bool:
        pass

# Composite interfaces for devices that support multiple operations
class Copier(ABC):
    @abstractmethod
    def copy_document(self) -> str:
        pass

# Implementations
class BasicPrinter(Printer):
    """Simple printer implementing only printing"""
    
    def print_document(self, document: str) -> bool:
        print(f"Basic printer: Printing {document}")
        return True

class BasicScanner(Scanner):
    """Simple scanner implementing only scanning"""
    
    def scan_document(self) -> str:
        print("Basic scanner: Scanning document...")
        return "scanned_document_data"

class NetworkPrinter(Printer, EmailSender):
    """Network printer with email capabilities"""
    
    def print_document(self, document: str) -> bool:
        print(f"Network printer: Printing {document}")
        return True
    
    def email_document(self, document: str, email: str) -> bool:
        print(f"Network printer: Emailing {document} to {email}")
        return True

class PhotoInkjetPrinter(Printer, PhotoPrinter):
    """Inkjet printer with photo printing capabilities"""
    
    def print_document(self, document: str) -> bool:
        print(f"Photo printer: Printing document {document}")
        return True
    
    def print_photos(self, photos: List[str]) -> bool:
        print(f"Photo printer: Printing {len(photos)} photos in high quality")
        return True

class SmartScanner(Scanner, CloudConnector):
    """Modern scanner with cloud connectivity"""
    
    def scan_document(self) -> str:
        print("Smart scanner: Scanning document...")
        return "high_resolution_scan_data"
    
    def scan_to_cloud(self, cloud_service: str) -> bool:
        print(f"Smart scanner: Uploading scan to {cloud_service}")
        return True

class OfficeAllInOne(Printer, Scanner, Copier, FaxMachine, EmailSender):
    """Office device implementing multiple interfaces"""
    
    def print_document(self, document: str) -> bool:
        print(f"All-in-one: Printing {document}")
        return True
    
    def scan_document(self) -> str:
        print("All-in-one: Scanning document...")
        return "office_scan_data"
    
    def copy_document(self) -> str:
        print("All-in-one: Making copy...")
        scanned = self.scan_document()
        self.print_document(scanned)
        return scanned
    
    def fax_document(self, document: str, number: str) -> bool:
        print(f"All-in-one: Faxing {document} to {number}")
        return True
    
    def email_document(self, document: str, email: str) -> bool:
        print(f"All-in-one: Emailing {document} to {email}")
        return True

class PremiumAllInOne(OfficeAllInOne, PhotoPrinter, CloudConnector):
    """Premium device with all features"""
    
    def print_photos(self, photos: List[str]) -> bool:
        print(f"Premium all-in-one: Printing {len(photos)} photos in premium quality")
        return True
    
    def scan_to_cloud(self, cloud_service: str) -> bool:
        print(f"Premium all-in-one: Uploading to {cloud_service} with OCR")
        return True

# Client code that works with specific interfaces
def basic_printing_job(printer: Printer, document: str):
    """Works with any printer"""
    print(f"Starting basic printing job...")
    printer.print_document(document)

def scanning_job(scanner: Scanner):
    """Works with any scanner"""
    print(f"Starting scanning job...")
    return scanner.scan_document()

def photo_printing_job(photo_printer: PhotoPrinter, photos: List[str]):
    """Works only with photo-capable printers"""
    print(f"Starting photo printing job...")
    photo_printer.print_photos(photos)

def email_job(email_device: EmailSender, document: str, recipient: str):
    """Works with any device that can send emails"""
    print(f"Starting email job...")
    email_device.email_document(document, recipient)

def copying_job(copier: Copier):
    """Works with any device that can copy"""
    print(f"Starting copying job...")
    return copier.copy_document()

def cloud_backup_job(cloud_device: CloudConnector, service: str):
    """Works with any cloud-enabled device"""
    print(f"Starting cloud backup job...")
    cloud_device.scan_to_cloud(service)

def fax_job(fax_machine: FaxMachine, document: str, number: str):
    """Works with any fax-capable device"""
    print(f"Starting fax job...")
    fax_machine.fax_document(document, number)

# Device management system
class DeviceManager:
    def __init__(self):
        self.devices = []
    
    def register_device(self, device):
        self.devices.append(device)
    
    def get_printers(self) -> List[Printer]:
        return [device for device in self.devices if isinstance(device, Printer)]
    
    def get_scanners(self) -> List[Scanner]:
        return [device for device in self.devices if isinstance(device, Scanner)]
    
    def get_email_devices(self) -> List[EmailSender]:
        return [device for device in self.devices if isinstance(device, EmailSender)]
    
    def get_photo_printers(self) -> List[PhotoPrinter]:
        return [device for device in self.devices if isinstance(device, PhotoPrinter)]
    
    def get_cloud_devices(self) -> List[CloudConnector]:
        return [device for device in self.devices if isinstance(device, CloudConnector)]

# Usage
manager = DeviceManager()

# Register various devices
manager.register_device(BasicPrinter())
manager.register_device(BasicScanner())
manager.register_device(NetworkPrinter())
manager.register_device(PhotoInkjetPrinter())
manager.register_device(SmartScanner())
manager.register_device(OfficeAllInOne())
manager.register_device(PremiumAllInOne())

# Use devices through appropriate interfaces
print("=== Basic Printing Jobs ===")
for printer in manager.get_printers():
    basic_printing_job(printer, "Test Document")

print("\n=== Scanning Jobs ===")
for scanner in manager.get_scanners():
    scanning_job(scanner)

print("\n=== Photo Printing Jobs ===")
photos = ["vacation1.jpg", "vacation2.jpg", "vacation3.jpg"]
for photo_printer in manager.get_photo_printers():
    photo_printing_job(photo_printer, photos)

print("\n=== Email Jobs ===")
for email_device in manager.get_email_devices():
    email_job(email_device, "Report", "manager@company.com")

print("\n=== Cloud Backup Jobs ===")
for cloud_device in manager.get_cloud_devices():
    cloud_backup_job(cloud_device, "Google Drive")
```

### Example 2: Media Player Interface

#### Violating ISP

```python
# BAD: Violates ISP - Fat media interface
from abc import ABC, abstractmethod

class MediaPlayer(ABC):
    """Fat interface forcing all players to support all media types and features"""
    
    @abstractmethod
    def play_audio(self, file_path: str) -> bool:
        pass
    
    @abstractmethod
    def play_video(self, file_path: str) -> bool:
        pass
    
    @abstractmethod
    def play_streaming_audio(self, url: str) -> bool:
        pass
    
    @abstractmethod
    def play_streaming_video(self, url: str) -> bool:
        pass
    
    @abstractmethod
    def record_audio(self, output_path: str) -> bool:
        pass
    
    @abstractmethod
    def record_video(self, output_path: str) -> bool:
        pass
    
    @abstractmethod
    def convert_format(self, input_path: str, output_path: str, format: str) -> bool:
        pass
    
    @abstractmethod
    def apply_audio_effects(self, effects: list) -> bool:
        pass
    
    @abstractmethod
    def apply_video_effects(self, effects: list) -> bool:
        pass
    
    @abstractmethod
    def create_playlist(self, files: list) -> bool:
        pass
    
    @abstractmethod
    def download_media(self, url: str, output_path: str) -> bool:
        pass

class SimpleAudioPlayer(MediaPlayer):
    """Simple audio player forced to implement video and advanced features"""
    
    def play_audio(self, file_path: str) -> bool:
        print(f"Playing audio: {file_path}")
        return True
    
    def play_video(self, file_path: str) -> bool:
        # Problem: Audio-only player doesn't support video
        raise NotImplementedError("This player doesn't support video")
    
    def play_streaming_audio(self, url: str) -> bool:
        # Problem: No network capabilities
        raise NotImplementedError("This player doesn't support streaming")
    
    def play_streaming_video(self, url: str) -> bool:
        raise NotImplementedError("This player doesn't support video or streaming")
    
    def record_audio(self, output_path: str) -> bool:
        # Problem: Player-only device, no recording
        raise NotImplementedError("This player doesn't support recording")
    
    def record_video(self, output_path: str) -> bool:
        raise NotImplementedError("This player doesn't support video or recording")
    
    def convert_format(self, input_path: str, output_path: str, format: str) -> bool:
        # Problem: No format conversion capabilities
        raise NotImplementedError("This player doesn't support format conversion")
    
    def apply_audio_effects(self, effects: list) -> bool:
        # Problem: Basic player, no effects
        raise NotImplementedError("This player doesn't support audio effects")
    
    def apply_video_effects(self, effects: list) -> bool:
        raise NotImplementedError("This player doesn't support video effects")
    
    def create_playlist(self, files: list) -> bool:
        # Problem: No playlist functionality
        raise NotImplementedError("This player doesn't support playlists")
    
    def download_media(self, url: str, output_path: str) -> bool:
        # Problem: No download capabilities
        raise NotImplementedError("This player doesn't support downloads")

class StreamingVideoPlayer(MediaPlayer):
    """Streaming video player forced to implement local files and recording"""
    
    def play_audio(self, file_path: str) -> bool:
        # Problem: Streaming-only, doesn't handle local files
        raise NotImplementedError("This player only supports streaming")
    
    def play_video(self, file_path: str) -> bool:
        # Problem: Streaming-only, doesn't handle local files
        raise NotImplementedError("This player only supports streaming")
    
    def play_streaming_audio(self, url: str) -> bool:
        print(f"Streaming audio from: {url}")
        return True
    
    def play_streaming_video(self, url: str) -> bool:
        print(f"Streaming video from: {url}")
        return True
    
    def record_audio(self, output_path: str) -> bool:
        # Problem: Streaming player doesn't record
        raise NotImplementedError("This player doesn't support recording")
    
    def record_video(self, output_path: str) -> bool:
        raise NotImplementedError("This player doesn't support recording")
    
    def convert_format(self, input_path: str, output_path: str, format: str) -> bool:
        # Problem: Streaming-only, no file conversion
        raise NotImplementedError("This player doesn't support format conversion")
    
    def apply_audio_effects(self, effects: list) -> bool:
        # Problem: Basic streaming, no effects
        raise NotImplementedError("This player doesn't support effects")
    
    def apply_video_effects(self, effects: list) -> bool:
        raise NotImplementedError("This player doesn't support effects")
    
    def create_playlist(self, files: list) -> bool:
        # Problem: Streaming URLs, not files
        raise NotImplementedError("This player uses streaming URLs, not playlists")
    
    def download_media(self, url: str, output_path: str) -> bool:
        # Problem: Streaming-only service
        raise NotImplementedError("This player doesn't support downloads")

# Client code that breaks with incomplete implementations
def media_production_workflow(player: MediaPlayer):
    """Expects full media production capabilities"""
    try:
        # Record content
        player.record_audio("intro.wav")
        player.record_video("main.mp4")
        
        # Apply effects
        player.apply_audio_effects(["noise_reduction", "normalize"])
        player.apply_video_effects(["color_correction", "stabilization"])
        
        # Convert formats
        player.convert_format("intro.wav", "intro.mp3", "mp3")
        
        # Create final playlist
        player.create_playlist(["intro.mp3", "main.mp4"])
        
    except NotImplementedError as e:
        print(f"Production workflow failed: {e}")

# Usage - demonstrates ISP violation
players = [
    SimpleAudioPlayer(),
    StreamingVideoPlayer()
]

for player in players:
    print(f"\nTesting {player.__class__.__name__}:")
    media_production_workflow(player)
```

#### Following ISP

```python
# GOOD: Following ISP - Segregated media interfaces
from abc import ABC, abstractmethod
from typing import List

# Core playback interfaces
class AudioPlayer(ABC):
    @abstractmethod
    def play_audio(self, file_path: str) -> bool:
        pass

class VideoPlayer(ABC):
    @abstractmethod
    def play_video(self, file_path: str) -> bool:
        pass

# Streaming interfaces
class StreamingAudioPlayer(ABC):
    @abstractmethod
    def play_streaming_audio(self, url: str) -> bool:
        pass

class StreamingVideoPlayer(ABC):
    @abstractmethod
    def play_streaming_video(self, url: str) -> bool:
        pass

# Recording interfaces
class AudioRecorder(ABC):
    @abstractmethod
    def record_audio(self, output_path: str) -> bool:
        pass

class VideoRecorder(ABC):
    @abstractmethod
    def record_video(self, output_path: str) -> bool:
        pass

# Processing interfaces
class MediaConverter(ABC):
    @abstractmethod
    def convert_format(self, input_path: str, output_path: str, format: str) -> bool:
        pass

class AudioEffectProcessor(ABC):
    @abstractmethod
    def apply_audio_effects(self, effects: List[str]) -> bool:
        pass

class VideoEffectProcessor(ABC):
    @abstractmethod
    def apply_video_effects(self, effects: List[str]) -> bool:
        pass

# Advanced features
class PlaylistManager(ABC):
    @abstractmethod
    def create_playlist(self, files: List[str]) -> bool:
        pass
    
    @abstractmethod
    def load_playlist(self, playlist_path: str) -> List[str]:
        pass

class MediaDownloader(ABC):
    @abstractmethod
    def download_media(self, url: str, output_path: str) -> bool:
        pass

# Implementations
class BasicAudioPlayer(AudioPlayer):
    """Simple audio player with only basic playback"""
    
    def play_audio(self, file_path: str) -> bool:
        print(f"Basic audio player: Playing {file_path}")
        return True

class BasicVideoPlayer(VideoPlayer):
    """Simple video player with only basic playback"""
    
    def play_video(self, file_path: str) -> bool:
        print(f"Basic video player: Playing {file_path}")
        return True

class NetworkAudioPlayer(AudioPlayer, StreamingAudioPlayer):
    """Audio player with streaming capabilities"""
    
    def play_audio(self, file_path: str) -> bool:
        print(f"Network audio player: Playing local file {file_path}")
        return True
    
    def play_streaming_audio(self, url: str) -> bool:
        print(f"Network audio player: Streaming from {url}")
        return True

class SmartTVPlayer(VideoPlayer, StreamingVideoPlayer, AudioPlayer, StreamingAudioPlayer):
    """Smart TV with comprehensive playback capabilities"""
    
    def play_audio(self, file_path: str) -> bool:
        print(f"Smart TV: Playing audio file {file_path}")
        return True
    
    def play_video(self, file_path: str) -> bool:
        print(f"Smart TV: Playing video file {file_path}")
        return True
    
    def play_streaming_audio(self, url: str) -> bool:
        print(f"Smart TV: Streaming audio from {url}")
        return True
    
    def play_streaming_video(self, url: str) -> bool:
        print(f"Smart TV: Streaming video from {url}")
        return True

class PodcastRecorder(AudioRecorder, AudioEffectProcessor):
    """Specialized device for podcast recording"""
    
    def record_audio(self, output_path: str) -> bool:
        print(f"Podcast recorder: Recording to {output_path}")
        return True
    
    def apply_audio_effects(self, effects: List[str]) -> bool:
        print(f"Podcast recorder: Applying effects {effects}")
        return True

class VideoProductionStudio(VideoPlayer, VideoRecorder, VideoEffectProcessor, MediaConverter):
    """Professional video production equipment"""
    
    def play_video(self, file_path: str) -> bool:
        print(f"Production studio: Previewing {file_path}")
        return True
    
    def record_video(self, output_path: str) -> bool:
        print(f"Production studio: Recording to {output_path}")
        return True
    
    def apply_video_effects(self, effects: List[str]) -> bool:
        print(f"Production studio: Applying professional effects {effects}")
        return True
    
    def convert_format(self, input_path: str, output_path: str, format: str) -> bool:
        print(f"Production studio: Converting {input_path} to {format}")
        return True

class MediaCenterPC(AudioPlayer, VideoPlayer, StreamingAudioPlayer, StreamingVideoPlayer, 
                   PlaylistManager, MediaDownloader):
    """Comprehensive media center"""
    
    def play_audio(self, file_path: str) -> bool:
        print(f"Media center: Playing audio {file_path}")
        return True
    
    def play_video(self, file_path: str) -> bool:
        print(f"Media center: Playing video {file_path}")
        return True
    
    def play_streaming_audio(self, url: str) -> bool:
        print(f"Media center: Streaming audio from {url}")
        return True
    
    def play_streaming_video(self, url: str) -> bool:
        print(f"Media center: Streaming video from {url}")
        return True
    
    def create_playlist(self, files: List[str]) -> bool:
        print(f"Media center: Creating playlist with {len(files)} items")
        return True
    
    def load_playlist(self, playlist_path: str) -> List[str]:
        print(f"Media center: Loading playlist from {playlist_path}")
        return ["song1.mp3", "song2.mp3", "video1.mp4"]
    
    def download_media(self, url: str, output_path: str) -> bool:
        print(f"Media center: Downloading {url} to {output_path}")
        return True

# Client functions for specific interfaces
def basic_audio_playback(player: AudioPlayer, files: List[str]):
    """Works with any audio player"""
    print("Starting audio playback session...")
    for file in files:
        player.play_audio(file)

def streaming_session(player: StreamingAudioPlayer, urls: List[str]):
    """Works with any streaming audio player"""
    print("Starting streaming session...")
    for url in urls:
        player.play_streaming_audio(url)

def video_playback_session(player: VideoPlayer, files: List[str]):
    """Works with any video player"""
    print("Starting video playback session...")
    for file in files:
        player.play_video(file)

def recording_session(recorder: AudioRecorder, output_files: List[str]):
    """Works with any audio recorder"""
    print("Starting recording session...")
    for output in output_files:
        recorder.record_audio(output)

def audio_production_workflow(processor: AudioEffectProcessor, effects: List[str]):
    """Works with any audio effect processor"""
    print("Starting audio production workflow...")
    processor.apply_audio_effects(effects)

def media_conversion_job(converter: MediaConverter, conversions: List[tuple]):
    """Works with any media converter"""
    print("Starting media conversion job...")
    for input_file, output_file, format_type in conversions:
        converter.convert_format(input_file, output_file, format_type)

def playlist_management(manager: PlaylistManager, files: List[str]):
    """Works with any playlist manager"""
    print("Managing playlists...")
    manager.create_playlist(files)
    loaded = manager.load_playlist("favorites.m3u")
    print(f"Loaded playlist with {len(loaded)} items")

def download_session(downloader: MediaDownloader, downloads: List[tuple]):
    """Works with any media downloader"""
    print("Starting download session...")
    for url, output_path in downloads:
        downloader.download_media(url, output_path)

# Media service that works with appropriate interfaces
class MediaService:
    def __init__(self):
        self.devices = []
    
    def register_device(self, device):
        self.devices.append(device)
    
    def get_audio_players(self) -> List[AudioPlayer]:
        return [d for d in self.devices if isinstance(d, AudioPlayer)]
    
    def get_video_players(self) -> List[VideoPlayer]:
        return [d for d in self.devices if isinstance(d, VideoPlayer)]
    
    def get_streaming_audio_players(self) -> List[StreamingAudioPlayer]:
        return [d for d in self.devices if isinstance(d, StreamingAudioPlayer)]
    
    def get_recorders(self) -> List[AudioRecorder]:
        return [d for d in self.devices if isinstance(d, AudioRecorder)]
    
    def get_converters(self) -> List[MediaConverter]:
        return [d for d in self.devices if isinstance(d, MediaConverter)]
    
    def get_playlist_managers(self) -> List[PlaylistManager]:
        return [d for d in self.devices if isinstance(d, PlaylistManager)]

# Usage
service = MediaService()

# Register various devices
service.register_device(BasicAudioPlayer())
service.register_device(BasicVideoPlayer())
service.register_device(NetworkAudioPlayer())
service.register_device(SmartTVPlayer())
service.register_device(PodcastRecorder())
service.register_device(VideoProductionStudio())
service.register_device(MediaCenterPC())

# Use devices through appropriate interfaces
print("=== Audio Playback ===")
audio_files = ["song1.mp3", "song2.mp3"]
for player in service.get_audio_players():
    basic_audio_playback(player, audio_files)

print("\n=== Streaming Audio ===")
streaming_urls = ["http://radio1.com/stream", "http://podcast.com/feed"]
for player in service.get_streaming_audio_players():
    streaming_session(player, streaming_urls)

print("\n=== Recording ===")
output_files = ["interview1.wav", "podcast_episode.wav"]
for recorder in service.get_recorders():
    recording_session(recorder, output_files)

print("\n=== Media Conversion ===")
conversions = [("audio.wav", "audio.mp3", "mp3"), ("video.avi", "video.mp4", "mp4")]
for converter in service.get_converters():
    media_conversion_job(converter, conversions)

print("\n=== Playlist Management ===")
playlist_files = ["song1.mp3", "song2.mp3", "song3.mp3"]
for manager in service.get_playlist_managers():
    playlist_management(manager, playlist_files)
```

### Example 3: Database Access Layer

#### Violating ISP

```python
# BAD: Violates ISP - Monolithic database interface
from abc import ABC, abstractmethod
from typing import Dict, List, Any, Optional

class DatabaseInterface(ABC):
    """Fat interface that forces all implementations to support all database operations"""
    
    # Basic CRUD operations
    @abstractmethod
    def create(self, table: str, data: Dict[str, Any]) -> int:
        pass
    
    @abstractmethod
    def read(self, table: str, id: int) -> Optional[Dict[str, Any]]:
        pass
    
    @abstractmethod
    def update(self, table: str, id: int, data: Dict[str, Any]) -> bool:
        pass
    
    @abstractmethod
    def delete(self, table: str, id: int) -> bool:
        pass
    
    # Query operations
    @abstractmethod
    def find_by_criteria(self, table: str, criteria: Dict[str, Any]) -> List[Dict[str, Any]]:
        pass
    
    @abstractmethod
    def execute_raw_sql(self, sql: str, params: tuple) -> List[Dict[str, Any]]:
        pass
    
    # Transaction management
    @abstractmethod
    def begin_transaction(self) -> None:
        pass
    
    @abstractmethod
    def commit_transaction(self) -> None:
        pass
    
    @abstractmethod
    def rollback_transaction(self) -> None:
        pass
    
    # Schema management
    @abstractmethod
    def create_table(self, table_name: str, schema: Dict[str, str]) -> bool:
        pass
    
    @abstractmethod
    def drop_table(self, table_name: str) -> bool:
        pass
    
    @abstractmethod
    def alter_table(self, table_name: str, changes: Dict[str, Any]) -> bool:
        pass
    
    # Performance and maintenance
    @abstractmethod
    def create_index(self, table: str, columns: List[str]) -> bool:
        pass
    
    @abstractmethod
    def analyze_performance(self, query: str) -> Dict[str, Any]:
        pass
    
    @abstractmethod
    def vacuum_database(self) -> bool:
        pass
    
    # Backup and restore
    @abstractmethod
    def backup_database(self, backup_path: str) -> bool:
        pass
    
    @abstractmethod
    def restore_database(self, backup_path: str) -> bool:
        pass

class ReadOnlyCache(DatabaseInterface):
    """Read-only cache forced to implement write operations"""
    
    def __init__(self):
        self.cache = {}
    
    def create(self, table: str, data: Dict[str, Any]) -> int:
        # Problem: Cache is read-only
        raise NotImplementedError("Cache is read-only")
    
    def read(self, table: str, id: int) -> Optional[Dict[str, Any]]:
        key = f"{table}:{id}"
        return self.cache.get(key)
    
    def update(self, table: str, id: int, data: Dict[str, Any]) -> bool:
        # Problem: Cache is read-only
        raise NotImplementedError("Cache is read-only")
    
    def delete(self, table: str, id: int) -> bool:
        # Problem: Cache is read-only
        raise NotImplementedError("Cache is read-only")
    
    def find_by_criteria(self, table: str, criteria: Dict[str, Any]) -> List[Dict[str, Any]]:
        # Problem: Cache doesn't support complex queries
        raise NotImplementedError("Cache doesn't support complex queries")
    
    def execute_raw_sql(self, sql: str, params: tuple) -> List[Dict[str, Any]]:
        # Problem: Cache doesn't support SQL
        raise NotImplementedError("Cache doesn't support SQL")
    
    def begin_transaction(self) -> None:
        # Problem: Cache doesn't support transactions
        raise NotImplementedError("Cache doesn't support transactions")
    
    def commit_transaction(self) -> None:
        raise NotImplementedError("Cache doesn't support transactions")
    
    def rollback_transaction(self) -> None:
        raise NotImplementedError("Cache doesn't support transactions")
    
    def create_table(self, table_name: str, schema: Dict[str, str]) -> bool:
        # Problem: Cache doesn't manage schemas
        raise NotImplementedError("Cache doesn't support schema management")
    
    def drop_table(self, table_name: str) -> bool:
        raise NotImplementedError("Cache doesn't support schema management")
    
    def alter_table(self, table_name: str, changes: Dict[str, Any]) -> bool:
        raise NotImplementedError("Cache doesn't support schema management")
    
    def create_index(self, table: str, columns: List[str]) -> bool:
        # Problem: Cache doesn't support indexing
        raise NotImplementedError("Cache doesn't support indexing")
    
    def analyze_performance(self, query: str) -> Dict[str, Any]:
        # Problem: Cache doesn't have performance analysis
        raise NotImplementedError("Cache doesn't support performance analysis")
    
    def vacuum_database(self) -> bool:
        # Problem: Cache doesn't need vacuuming
        raise NotImplementedError("Cache doesn't support vacuuming")
    
    def backup_database(self, backup_path: str) -> bool:
        # Problem: Cache doesn't support backup
        raise NotImplementedError("Cache doesn't support backup")
    
    def restore_database(self, backup_path: str) -> bool:
        # Problem: Cache doesn't support restore
        raise NotImplementedError("Cache doesn't support restore")

class SimpleFileStorage(DatabaseInterface):
    """Simple file storage forced to implement database features"""
    
    def __init__(self, file_path: str):
        self.file_path = file_path
        self.data = {}
    
    def create(self, table: str, data: Dict[str, Any]) -> int:
        # Simple implementation
        if table not in self.data:
            self.data[table] = {}
        new_id = len(self.data[table]) + 1
        self.data[table][new_id] = data
        return new_id
    
    def read(self, table: str, id: int) -> Optional[Dict[str, Any]]:
        return self.data.get(table, {}).get(id)
    
    def update(self, table: str, id: int, data: Dict[str, Any]) -> bool:
        if table in self.data and id in self.data[table]:
            self.data[table][id].update(data)
            return True
        return False
    
    def delete(self, table: str, id: int) -> bool:
        if table in self.data and id in self.data[table]:
            del self.data[table][id]
            return True
        return False
    
    def find_by_criteria(self, table: str, criteria: Dict[str, Any]) -> List[Dict[str, Any]]:
        # Problem: File storage doesn't support complex queries
        raise NotImplementedError("File storage doesn't support complex queries")
    
    def execute_raw_sql(self, sql: str, params: tuple) -> List[Dict[str, Any]]:
        # Problem: File storage doesn't support SQL
        raise NotImplementedError("File storage doesn't support SQL")
    
    def begin_transaction(self) -> None:
        # Problem: File storage doesn't support transactions
        raise NotImplementedError("File storage doesn't support transactions")
    
    def commit_transaction(self) -> None:
        raise NotImplementedError("File storage doesn't support transactions")
    
    def rollback_transaction(self) -> None:
        raise NotImplementedError("File storage doesn't support transactions")
    
    def create_table(self, table_name: str, schema: Dict[str, str]) -> bool:
        # Problem: File storage doesn't enforce schemas
        raise NotImplementedError("File storage doesn't support schemas")
    
    def drop_table(self, table_name: str) -> bool:
        if table_name in self.data:
            del self.data[table_name]
            return True
        return False
    
    def alter_table(self, table_name: str, changes: Dict[str, Any]) -> bool:
        raise NotImplementedError("File storage doesn't support schema changes")
    
    def create_index(self, table: str, columns: List[str]) -> bool:
        # Problem: File storage doesn't support indexing
        raise NotImplementedError("File storage doesn't support indexing")
    
    def analyze_performance(self, query: str) -> Dict[str, Any]:
        # Problem: File storage doesn't have performance analysis
        raise NotImplementedError("File storage doesn't support performance analysis")
    
    def vacuum_database(self) -> bool:
        # Problem: File storage doesn't need vacuuming
        raise NotImplementedError("File storage doesn't need vacuuming")
    
    def backup_database(self, backup_path: str) -> bool:
        # Problem: File storage doesn't have built-in backup
        raise NotImplementedError("File storage doesn't support automatic backup")
    
    def restore_database(self, backup_path: str) -> bool:
        raise NotImplementedError("File storage doesn't support automatic restore")

# Client code that expects full database functionality
def database_admin_operations(db: DatabaseInterface):
    """Expects full database administration capabilities"""
    try:
        # Schema management
        db.create_table("users", {"id": "int", "name": "string", "email": "string"})
        db.create_index("users", ["email"])
        
        # Performance analysis
        performance = db.analyze_performance("SELECT * FROM users WHERE email = ?")
        print(f"Query performance: {performance}")
        
        # Database maintenance
        db.vacuum_database()
        
        # Backup
        db.backup_database("/backups/users_backup.sql")
        
    except NotImplementedError as e:
        print(f"Admin operation failed: {e}")

# Usage - demonstrates ISP violation
databases = [
    ReadOnlyCache(),
    SimpleFileStorage("data.json")
]

for db in databases:
    print(f"\nTesting admin operations on {db.__class__.__name__}:")
    database_admin_operations(db)
```

#### Following ISP

```python
# GOOD: Following ISP - Segregated database interfaces
from abc import ABC, abstractmethod
from typing import Dict, List, Any, Optional

# Basic data access interfaces
class DataReader(ABC):
    @abstractmethod
    def read(self, table: str, id: int) -> Optional[Dict[str, Any]]:
        pass

class DataWriter(ABC):
    @abstractmethod
    def create(self, table: str, data: Dict[str, Any]) -> int:
        pass
    
    @abstractmethod
    def update(self, table: str, id: int, data: Dict[str, Any]) -> bool:
        pass
    
    @abstractmethod
    def delete(self, table: str, id: int) -> bool:
        pass

# Query interfaces
class QueryEngine(ABC):
    @abstractmethod
    def find_by_criteria(self, table: str, criteria: Dict[str, Any]) -> List[Dict[str, Any]]:
        pass

class SQLExecutor(ABC):
    @abstractmethod
    def execute_raw_sql(self, sql: str, params: tuple) -> List[Dict[str, Any]]:
        pass

# Transaction interfaces
class TransactionManager(ABC):
    @abstractmethod
    def begin_transaction(self) -> None:
        pass
    
    @abstractmethod
    def commit_transaction(self) -> None:
        pass
    
    @abstractmethod
    def rollback_transaction(self) -> None:
        pass

# Schema management interfaces
class SchemaManager(ABC):
    @abstractmethod
    def create_table(self, table_name: str, schema: Dict[str, str]) -> bool:
        pass
    
    @abstractmethod
    def drop_table(self, table_name: str) -> bool:
        pass
    
    @abstractmethod
    def alter_table(self, table_name: str, changes: Dict[str, Any]) -> bool:
        pass

# Performance interfaces
class IndexManager(ABC):
    @abstractmethod
    def create_index(self, table: str, columns: List[str]) -> bool:
        pass
    
    @abstractmethod
    def drop_index(self, table: str, index_name: str) -> bool:
        pass

class PerformanceAnalyzer(ABC):
    @abstractmethod
    def analyze_performance(self, query: str) -> Dict[str, Any]:
        pass

# Maintenance interfaces
class DatabaseMaintenance(ABC):
    @abstractmethod
    def vacuum_database(self) -> bool:
        pass
    
    @abstractmethod
    def optimize_tables(self) -> bool:
        pass

# Backup interfaces
class BackupManager(ABC):
    @abstractmethod
    def backup_database(self, backup_path: str) -> bool:
        pass
    
    @abstractmethod
    def restore_database(self, backup_path: str) -> bool:
        pass

# Implementations
class MemoryCache(DataReader):
    """Simple read-only cache"""
    
    def __init__(self):
        self.cache = {}
    
    def read(self, table: str, id: int) -> Optional[Dict[str, Any]]:
        key = f"{table}:{id}"
        return self.cache.get(key)
    
    def set(self, table: str, id: int, data: Dict[str, Any]):
        """Cache-specific method not part of interface"""
        key = f"{table}:{id}"
        self.cache[key] = data
    
    def evict(self, table: str, id: int):
        """Cache-specific method"""
        key = f"{table}:{id}"
        self.cache.pop(key, None)

class SimpleFileStore(DataReader, DataWriter):
    """Simple file-based storage with basic CRUD"""
    
    def __init__(self, file_path: str):
        self.file_path = file_path
        self.data = {}
    
    def create(self, table: str, data: Dict[str, Any]) -> int:
        if table not in self.data:
            self.data[table] = {}
        new_id = len(self.data[table]) + 1
        self.data[table][new_id] = data
        return new_id
    
    def read(self, table: str, id: int) -> Optional[Dict[str, Any]]:
        return self.data.get(table, {}).get(id)
    
    def update(self, table: str, id: int, data: Dict[str, Any]) -> bool:
        if table in self.data and id in self.data[table]:
            self.data[table][id].update(data)
            return True
        return False
    
    def delete(self, table: str, id: int) -> bool:
        if table in self.data and id in self.data[table]:
            del self.data[table][id]
            return True
        return False

class InMemoryDatabase(DataReader, DataWriter, QueryEngine, TransactionManager):
    """In-memory database with transactions and queries"""
    
    def __init__(self):
        self.data = {}
        self.transaction_log = []
        self.in_transaction = False
    
    def create(self, table: str, data: Dict[str, Any]) -> int:
        if table not in self.data:
            self.data[table] = {}
        new_id = len(self.data[table]) + 1
        self.data[table][new_id] = data.copy()
        if self.in_transaction:
            self.transaction_log.append(('create', table, new_id, data.copy()))
        return new_id
    
    def read(self, table: str, id: int) -> Optional[Dict[str, Any]]:
        return self.data.get(table, {}).get(id)
    
    def update(self, table: str, id: int, data: Dict[str, Any]) -> bool:
        if table in self.data and id in self.data[table]:
            if self.in_transaction:
                old_data = self.data[table][id].copy()
                self.transaction_log.append(('update', table, id, old_data))
            self.data[table][id].update(data)
            return True
        return False
    
    def delete(self, table: str, id: int) -> bool:
        if table in self.data and id in self.data[table]:
            if self.in_transaction:
                old_data = self.data[table][id].copy()
                self.transaction_log.append(('delete', table, id, old_data))
            del self.data[table][id]
            return True
        return False
    
    def find_by_criteria(self, table: str, criteria: Dict[str, Any]) -> List[Dict[str, Any]]:
        if table not in self.data:
            return []
        
        results = []
        for id, record in self.data[table].items():
            match = True
            for key, value in criteria.items():
                if key not in record or record[key] != value:
                    match = False
                    break
            if match:
                results.append({**record, 'id': id})
        return results
    
    def begin_transaction(self) -> None:
        if self.in_transaction:
            raise RuntimeError("Transaction already in progress")
        self.in_transaction = True
        self.transaction_log = []
    
    def commit_transaction(self) -> None:
        if not self.in_transaction:
            raise RuntimeError("No transaction in progress")
        self.in_transaction = False
        self.transaction_log = []
    
    def rollback_transaction(self) -> None:
        if not self.in_transaction:
            raise RuntimeError("No transaction in progress")
        
        # Rollback changes in reverse order
        for operation, table, id, data in reversed(self.transaction_log):
            if operation == 'create':
                if table in self.data and id in self.data[table]:
                    del self.data[table][id]
            elif operation == 'update':
                if table in self.data:
                    self.data[table][id] = data
            elif operation == 'delete':
                if table not in self.data:
                    self.data[table] = {}
                self.data[table][id] = data
        
        self.in_transaction = False
        self.transaction_log = []

class FullSQLDatabase(DataReader, DataWriter, QueryEngine, SQLExecutor, 
                     TransactionManager, SchemaManager, IndexManager, 
                     PerformanceAnalyzer, DatabaseMaintenance, BackupManager):
    """Full-featured SQL database implementation"""
    
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        self.schemas = {}
        self.indexes = {}
        self.data = {}
        self.in_transaction = False
    
    # DataReader/DataWriter implementation
    def create(self, table: str, data: Dict[str, Any]) -> int:
        print(f"SQL DB: Creating record in {table}")
        if table not in self.data:
            self.data[table] = {}
        new_id = len(self.data[table]) + 1
        self.data[table][new_id] = data
        return new_id
    
    def read(self, table: str, id: int) -> Optional[Dict[str, Any]]:
        print(f"SQL DB: Reading from {table} id {id}")
        return self.data.get(table, {}).get(id)
    
    def update(self, table: str, id: int, data: Dict[str, Any]) -> bool:
        print(f"SQL DB: Updating {table} id {id}")
        if table in self.data and id in self.data[table]:
            self.data[table][id].update(data)
            return True
        return False
    
    def delete(self, table: str, id: int) -> bool:
        print(f"SQL DB: Deleting from {table} id {id}")
        if table in self.data and id in self.data[table]:
            del self.data[table][id]
            return True
        return False
    
    # QueryEngine implementation
    def find_by_criteria(self, table: str, criteria: Dict[str, Any]) -> List[Dict[str, Any]]:
        print(f"SQL DB: Querying {table} with criteria {criteria}")
        # Implementation similar to InMemoryDatabase
        return []
    
    # SQLExecutor implementation
    def execute_raw_sql(self, sql: str, params: tuple) -> List[Dict[str, Any]]:
        print(f"SQL DB: Executing SQL: {sql}")
        return []
    
    # TransactionManager implementation
    def begin_transaction(self) -> None:
        print("SQL DB: Beginning transaction")
        self.in_transaction = True
    
    def commit_transaction(self) -> None:
        print("SQL DB: Committing transaction")
        self.in_transaction = False
    
    def rollback_transaction(self) -> None:
        print("SQL DB: Rolling back transaction")
        self.in_transaction = False
    
    # SchemaManager implementation
    def create_table(self, table_name: str, schema: Dict[str, str]) -> bool:
        print(f"SQL DB: Creating table {table_name} with schema {schema}")
        self.schemas[table_name] = schema
        return True
    
    def drop_table(self, table_name: str) -> bool:
        print(f"SQL DB: Dropping table {table_name}")
        self.schemas.pop(table_name, None)
        self.data.pop(table_name, None)
        return True
    
    def alter_table(self, table_name: str, changes: Dict[str, Any]) -> bool:
        print(f"SQL DB: Altering table {table_name}")
        return True
    
    # IndexManager implementation
    def create_index(self, table: str, columns: List[str]) -> bool:
        print(f"SQL DB: Creating index on {table} columns {columns}")
        if table not in self.indexes:
            self.indexes[table] = []
        self.indexes[table].append(columns)
        return True
    
    def drop_index(self, table: str, index_name: str) -> bool:
        print(f"SQL DB: Dropping index {index_name} on {table}")
        return True
    
    # PerformanceAnalyzer implementation
    def analyze_performance(self, query: str) -> Dict[str, Any]:
        print(f"SQL DB: Analyzing performance of query: {query}")
        return {"execution_time": 0.05, "rows_examined": 100, "cost": 1.2}
    
    # DatabaseMaintenance implementation
    def vacuum_database(self) -> bool:
        print("SQL DB: Vacuuming database")
        return True
    
    def optimize_tables(self) -> bool:
        print("SQL DB: Optimizing tables")
        return True
    
    # BackupManager implementation
    def backup_database(self, backup_path: str) -> bool:
        print(f"SQL DB: Backing up to {backup_path}")
        return True
    
    def restore_database(self, backup_path: str) -> bool:
        print(f"SQL DB: Restoring from {backup_path}")
        return True

# Client functions that work with specific interfaces
def basic_data_operations(reader: DataReader, writer: DataWriter):
    """Works with any data reader/writer"""
    print("Performing basic data operations...")
    
    # Create data
    user_id = writer.create("users", {"name": "Alice", "email": "alice@example.com"})
    print(f"Created user with ID: {user_id}")
    
    # Read data
    user = reader.read("users", user_id)
    print(f"Read user: {user}")
    
    # Update data
    writer.update("users", user_id, {"email": "alice.smith@example.com"})
    
    # Read updated data
    updated_user = reader.read("users", user_id)
    print(f"Updated user: {updated_user}")

def query_operations(query_engine: QueryEngine):
    """Works with any query engine"""
    print("Performing query operations...")
    results = query_engine.find_by_criteria("users", {"name": "Alice"})
    print(f"Query results: {results}")

def transactional_operations(tx_manager: TransactionManager, writer: DataWriter):
    """Works with any transaction manager"""
    print("Performing transactional operations...")
    
    tx_manager.begin_transaction()
    try:
        writer.create("orders", {"user_id": 1, "amount": 100.0})
        writer.create("orders", {"user_id": 1, "amount": 50.0})
        tx_manager.commit_transaction()
        print("Transaction committed successfully")
    except Exception as e:
        tx_manager.rollback_transaction()
        print(f"Transaction rolled back: {e}")

def schema_operations(schema_manager: SchemaManager):
    """Works with any schema manager"""
    print("Performing schema operations...")
    
    schema_manager.create_table("products", {
        "id": "int",
        "name": "varchar(255)",
        "price": "decimal(10,2)"
    })
    print("Table created successfully")

def performance_operations(analyzer: PerformanceAnalyzer, index_manager: IndexManager):
    """Works with performance analysis tools"""
    print("Performing performance operations...")
    
    index_manager.create_index("users", ["email"])
    performance = analyzer.analyze_performance("SELECT * FROM users WHERE email = 'alice@example.com'")
    print(f"Performance analysis: {performance}")

def maintenance_operations(maintenance: DatabaseMaintenance):
    """Works with database maintenance tools"""
    print("Performing maintenance operations...")
    maintenance.vacuum_database()
    maintenance.optimize_tables()

def backup_operations(backup_manager: BackupManager):
    """Works with backup managers"""
    print("Performing backup operations...")
    backup_manager.backup_database("/backups/mydb_backup.sql")

# Database service that manages different implementations
class DatabaseService:
    def __init__(self):
        self.connections = {}
    
    def register_connection(self, name: str, connection):
        self.connections[name] = connection
    
    def get_readers(self) -> List[DataReader]:
        return [conn for conn in self.connections.values() if isinstance(conn, DataReader)]
    
    def get_writers(self) -> List[DataWriter]:
        return [conn for conn in self.connections.values() if isinstance(conn, DataWriter)]
    
    def get_query_engines(self) -> List[QueryEngine]:
        return [conn for conn in self.connections.values() if isinstance(conn, QueryEngine)]
    
    def get_transaction_managers(self) -> List[TransactionManager]:
        return [conn for conn in self.connections.values() if isinstance(conn, TransactionManager)]

# Usage
service = DatabaseService()

# Register different database implementations
service.register_connection("cache", MemoryCache())
service.register_connection("file_store", SimpleFileStore("data.json"))
service.register_connection("memory_db", InMemoryDatabase())
service.register_connection("sql_db", FullSQLDatabase("postgresql://localhost/mydb"))

# Use appropriate interfaces
print("=== Basic Data Operations ===")
for i, reader in enumerate(service.get_readers()):
    writers = service.get_writers()
    if i < len(writers):
        basic_data_operations(reader, writers[i])

print("\n=== Query Operations ===")
for query_engine in service.get_query_engines():
    query_operations(query_engine)

print("\n=== Transactional Operations ===")
for tx_manager in service.get_transaction_managers():
    writers = service.get_writers()
    if writers:
        transactional_operations(tx_manager, writers[0])
```

## Interface Design Patterns

### 1. Role Interfaces
Design interfaces around the roles that clients need to play, not around the capabilities of implementing classes.

```python
# Good: Role-based interfaces
class OrderProcessor(ABC):
    @abstractmethod
    def process_order(self, order: Order) -> bool:
        pass

class PaymentHandler(ABC):
    @abstractmethod
    def process_payment(self, payment: Payment) -> bool:
        pass

class InventoryChecker(ABC):
    @abstractmethod
    def check_availability(self, item: str, quantity: int) -> bool:
        pass
```

### 2. Composition of Interfaces
Combine small interfaces to create more complex contracts when needed.

```python
class ECommerceService(OrderProcessor, PaymentHandler, InventoryChecker):
    """Composite interface for full e-commerce functionality"""
    pass
```

### 3. Protocol-Based Design (Python 3.8+)
Use Python's Protocol for structural subtyping.

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

class Printable(Protocol):
    def print(self) -> None: ...

# Any class with these methods automatically satisfies the protocol
class Document:
    def draw(self) -> None:
        print("Drawing document")
    
    def print(self) -> None:
        print("Printing document")

def render_item(item: Drawable) -> None:
    item.draw()
```

## Benefits of Following ISP

### 1. **Reduced Coupling**
Clients only depend on the methods they actually use.

### 2. **Improved Testability**
Smaller interfaces are easier to mock and test.

### 3. **Enhanced Flexibility**
Classes can implement only the interfaces they need.

### 4. **Better Code Organization**
Clear separation of concerns and responsibilities.

### 5. **Easier Maintenance**
Changes to one interface don't affect unrelated clients.

### 6. **Increased Reusability**
Small, focused interfaces are more likely to be reused.

## Common Pitfalls

### 1. **Interface Bloat**
Adding too many methods to a single interface over time.

### 2. **God Interfaces**
Creating interfaces that try to handle everything.

### 3. **Premature Interface Creation**
Creating interfaces before understanding actual client needs.

### 4. **Over-Segregation**
Creating too many tiny interfaces that don't provide meaningful abstractions.

### 5. **Ignoring Client Needs**
Designing interfaces based on implementation rather than client requirements.

### 6. **Mixing Abstraction Levels**
Combining high-level and low-level operations in the same interface.

## Best Practices

### 1. **Start with Client Needs**
Design interfaces based on what clients actually need to do.

### 2. **Keep Interfaces Focused**
Each interface should represent a single, cohesive concept or role.

### 3. **Use Composition**
Combine small interfaces to create larger contracts when needed.

### 4. **Favor Many Small Interfaces**
Prefer multiple small interfaces over one large interface.

### 5. **Regular Interface Review**
Regularly review interfaces to ensure they remain focused and relevant.

### 6. **Use Python's Type System**
Leverage type hints, Protocols, and abstract base classes effectively.

### 7. **Document Interface Contracts**
Clearly document what each interface represents and how it should be used.

### 8. **Consider Future Extensions**
Design interfaces to be extensible without breaking existing clients.

## Measuring ISP Compliance

### Code Metrics to Watch:
- **Interface Size**: Number of methods per interface
- **Implementation Completeness**: Percentage of interface methods actually used by implementations
- **Client Dependencies**: Number of interface methods actually used by each client
- **Empty Implementations**: Number of methods that throw "not implemented" exceptions

### Questions to Ask:
1. Are there methods in this interface that some implementations don't need?
2. Do clients use all the methods of the interfaces they depend on?
3. Can I split this interface into smaller, more focused interfaces?
4. Are there implementations that leave methods empty or throw exceptions?
5. Do changes to one part of an interface affect unrelated clients?

### ISP Validation Checklist:
- [ ] Each interface represents a single, cohesive role or responsibility
- [ ] All implementations actually use all methods of the interfaces they implement
- [ ] No client is forced to depend on methods it doesn't use
- [ ] Interfaces are designed around client needs, not implementation capabilities
- [ ] Large interfaces are composed of smaller, focused interfaces
- [ ] No interface methods consistently throw "not implemented" exceptions
- [ ] Changes to interface methods only affect clients that actually use those methods

## Conclusion

The Interface Segregation Principle is essential for creating maintainable and flexible systems with well-designed interfaces. By keeping interfaces small and focused on specific client needs, you create systems that are easier to understand, test, and maintain.

ISP encourages you to think from the client's perspective when designing interfaces, ensuring that each interface serves a specific purpose and doesn't burden clients with unnecessary dependencies. This leads to more modular designs where components can be easily composed, tested, and evolved independently.

Remember that ISP is not about creating an interface for every method - it's about creating meaningful abstractions that serve specific client roles. The goal is to find the right balance between having interfaces that are focused enough to be useful but substantial enough to represent meaningful contracts.

By following ISP, you create interfaces that are more likely to be stable over time, easier to implement, and more pleasant to work with. This ultimately leads to codebases that are more maintainable, testable, and adaptable to changing requirements.

The key to successful ISP implementation is to always consider the clients first: what do they need to accomplish, and how can you provide interfaces that support their goals without forcing them to depend on irrelevant functionality?