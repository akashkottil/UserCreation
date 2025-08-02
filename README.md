# User Tracking System for D2Flight iOS App

A comprehensive user tracking and analytics system for iOS applications built with SwiftUI. This system handles user creation, session management, and event tracking with automatic fallback mechanisms and robust error handling.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [API Reference](#api-reference)
- [Event Types](#event-types)
- [Best Practices](#best-practices)
- [Debugging](#debugging)
- [Troubleshooting](#troubleshooting)

## 🔍 Overview

This tracking system provides:
- **Automatic User Creation**: Creates unique users with device-specific identifiers
- **Session Management**: Tracks user sessions and events throughout the app lifecycle
- **Event Tracking**: Comprehensive event tracking for user interactions
- **Fallback Mechanisms**: Handles IDFA restrictions with UUID fallbacks
- **Debug Support**: Extensive logging for development and debugging

## 🏗 Architecture

The system consists of several key components:

```
├── Models/
│   └── UserModels.swift          # Data models for API requests/responses
├── Networks/
│   └── UserApi.swift             # API service layer
├── ViewModels/
│   └── UserManager.swift         # Main business logic and state management
├── Extensions/
│   └── UserTrack.swift           # SwiftUI view extensions for easy tracking
├── UserConfig.swift              # Configuration constants and utilities
└── README.md                     # App entry point setup
```

## 🚀 Installation

### 1. Add Required Dependencies

Add to your `Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.0.0")
]
```

### 2. Add Files to Your Project

Copy all the provided files to your Xcode project:
- `Models/UserModels.swift`
- `Networks/UserApi.swift`
- `ViewModels/UserManager.swift`
- `Extensions/UserTrack.swift`
- `UserConfig.swift`

### 3. Update Your App Entry Point

Replace your app's main structure with the provided setup:

```swift
import SwiftUI

@main
struct YourApp: App {
    @StateObject private var userManager = UserManager.shared
    
    init() {
        setupUserTracking()
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(userManager)
                .onReceive(NotificationCenter.default.publisher(for: UIApplication.didBecomeActiveNotification)) { _ in
                    UserManager.shared.createSession(eventType: .appLaunch, vertical: .flight)
                }
        }
    }
    
    private func setupUserTracking() {
        UserManager.shared.initializeUser()
    }
}
```

## ⚙️ Configuration

### 1. Update UserConfig.swift

Modify the configuration constants for your app:

```swift
struct UserConfig {
    // Update these for your app
    static let appCode = "your_app_code"
    static let appName = "YourAppName"
    static let appVersion = "1.0.0"
    
    // Update API endpoint
    static let defaultCountryCode = "US" // or your target country
    
    // Add any app-specific configuration
}
```

### 2. Update API Base URL

In `UserApi.swift`, update the base URL:

```swift
private let baseURL = "https://your-api-domain.com/api/v1"
```

## 📱 Usage

### Basic Event Tracking

#### Method 1: Using View Extensions (Recommended)

```swift
import SwiftUI

struct SearchView: View {
    var body: some View {
        VStack {
            Button("Search Flights") {
                // Your search logic here
            }
            .trackUserEvent(.searchButtonClick, vertical: .flight, tag: "main_search")
            
            // For simple button tracking
            Button("Apply Filter") {
                // Filter logic
            }
            .onTapGesture {
                UserManager.shared.trackButtonTap(.filterApplied, vertical: .flight, tag: "price_filter")
            }
        }
    }
}
```

#### Method 2: Direct UserManager Calls

```swift
import SwiftUI

struct FlightSearchView: View {
    @EnvironmentObject var userManager: UserManager
    
    var body: some View {
        VStack {
            Button("Search Flights") {
                // Track the search action
                userManager.createSession(
                    eventType: .flightSearch,
                    vertical: .flight,
                    tag: "flight_search_button",
                    additionalData: [
                        "origin": "NYC",
                        "destination": "LAX",
                        "passengers": "2"
                    ]
                )
                
                // Your search logic here
                performFlightSearch()
            }
        }
    }
    
    private func performFlightSearch() {
        // Your search implementation
    }
}
```

### Convenience Methods

The system provides convenience methods for common tracking scenarios:

```swift
// Track search button clicks
UserManager.shared.trackSearchButtonClick(vertical: .flight)

// Track specific search types
UserManager.shared.trackFlightSearch()
UserManager.shared.trackHotelSearch()
UserManager.shared.trackRentalSearch()

// Track location selection
UserManager.shared.createSession(
    eventType: .locationSelected,
    vertical: .flight,
    tag: "origin_selection"
)

// Track date selection
UserManager.shared.createSession(
    eventType: .dateSelected,
    vertical: .flight,
    tag: "departure_date"
)
```

### Advanced Tracking with Additional Data

```swift
UserManager.shared.createSession(
    eventType: .adClick,
    vertical: .flight,
    tag: "banner_ad",
    additionalData: [
        "ad_id": "12345",
        "campaign_id": "summer_sale",
        "adgroup_id": "flights_usa",
        "ad_objective_name": "conversions"
    ]
)
```

## 📚 API Reference

### UserManager

The main class for managing user tracking.

#### Properties

```swift
@Published var userId: Int?           // Current user ID
@Published var currentSessionId: Int? // Current session ID
@Published var isUserCreated: Bool    // Whether user exists
```

#### Methods

```swift
// Initialize user (call on app launch)
func initializeUser()

// Create session for events
func createSession(
    eventType: UserEventType,
    vertical: UserVertical = .flight,
    tag: String? = nil,
    additionalData: [String: String]? = nil
)

// Convenience methods
func trackSearchButtonClick(vertical: UserVertical = .flight)
func trackFlightSearch()
func trackHotelSearch()
func trackRentalSearch()

// Utility methods
var isValidUser: Bool { get }
var installDate: Date? { get }
func clearUserData()
func getAppInfo() -> [String: Any]
```

### View Extensions

```swift
// Track user events with tap gesture
func trackUserEvent(
    _ eventType: UserEventType,
    vertical: UserVertical = .flight,
    tag: String? = nil,
    additionalData: [String: String]? = nil
) -> some View

// Track button taps directly
func trackButtonTap(
    _ eventType: UserEventType,
    vertical: UserVertical = .flight,
    tag: String? = nil
)
```

## 🎯 Event Types

### Available Event Types

```swift
enum UserEventType: String, CaseIterable {
    case searchButtonClick = "search_button_click"
    case adClick = "ad_click"
    case appLaunch = "app_launch"
    case flightSearch = "flight_search"
    case hotelSearch = "hotel_search"
    case rentalSearch = "rental_search"
    case filterApplied = "filter_applied"
    case resultSelected = "result_selected"
    case bookingAttempt = "booking_attempt"
    case locationSelected = "location_selected"
    case dateSelected = "date_selected"
}
```

### Available Verticals

```swift
enum UserVertical: String, CaseIterable {
    case flight = "flight"
    case hotel = "hotel"
    case car = "car"
    case general = "general"
}
```

## 🎯 Best Practices

### 1. Use Meaningful Tags

```swift
// Good
.trackUserEvent(.searchButtonClick, tag: "homepage_search")
.trackUserEvent(.filterApplied, tag: "price_range_filter")

// Avoid generic tags
.trackUserEvent(.searchButtonClick, tag: "button")
```

### 2. Include Relevant Additional Data

```swift
UserManager.shared.createSession(
    eventType: .flightSearch,
    vertical: .flight,
    tag: "search_executed",
    additionalData: [
        "origin": origin,
        "destination": destination,
        "departure_date": departureDate,
        "return_date": returnDate,
        "passengers": "\(passengerCount)",
        "class": selectedClass
    ]
)
```

### 3. Track Key User Journey Events

```swift
// App lifecycle
UserManager.shared.createSession(eventType: .appLaunch, vertical: .general)

// User interactions
UserManager.shared.createSession(eventType: .locationSelected, vertical: .flight)
UserManager.shared.createSession(eventType: .dateSelected, vertical: .flight)
UserManager.shared.createSession(eventType: .searchButtonClick, vertical: .flight)

// Results and conversions
UserManager.shared.createSession(eventType: .resultSelected, vertical: .flight)
UserManager.shared.createSession(eventType: .bookingAttempt, vertical: .flight)
```

### 4. Handle Different Verticals

```swift
// Flight-specific tracking
UserManager.shared.trackFlightSearch()

// Hotel-specific tracking
UserManager.shared.trackHotelSearch()

// Car rental-specific tracking
UserManager.shared.trackRentalSearch()
```

## 🐛 Debugging

### Enable Debug Logging

Debug logging is automatically enabled in debug builds. In `UserConfig.swift`:

```swift
#if DEBUG
static let isDebugMode = true
static let enableVerboseLogging = true
#else
static let isDebugMode = false
static let enableVerboseLogging = false
#endif
```

### Debug Console Output

When debugging is enabled, you'll see detailed logs:

```
👤 Creating user with request:
   Device ID: ABC123-DEF456-GHI789
   App: d1_ios_flight
   Vendor ID: VENDOR-UUID
   Pseudo ID: abc123def456ghi789jkl

✅ User created successfully!
   User ID: 12345
   Message: User created successfully

📊 Creating session for user 12345:
   Type: api
   Vertical: flight
   Country: IN
   Route: unknown

✅ Session created successfully!
   Session ID: 67890
```

### Check User Status

```swift
// Check if user is properly initialized
if UserManager.shared.isValidUser {
    print("User is valid: ID \(UserManager.shared.userId ?? -1)")
} else {
    print("User initialization failed")
}

// Get app information
let appInfo = UserManager.shared.getAppInfo()
print("App Info: \(appInfo)")
```

## 🔧 Troubleshooting

### Common Issues

#### 1. User Creation Failed

**Problem**: API returns error during user creation

**Solutions**:
- Check API endpoint URL in `UserApi.swift`
- Verify network connectivity
- Check API key/authentication if required
- Review server logs for detailed error information

#### 2. IDFA Not Available

**Problem**: Device ID shows as all zeros

**Solution**: The system automatically falls back to UUID
```swift
// This is handled automatically
if deviceId == "00000000-0000-0000-0000-000000000000" {
    deviceIdType = UserConfig.fallbackDeviceIdType
    deviceId = UUID().uuidString
}
```

#### 3. Events Not Tracking

**Problem**: Events aren't being sent to the server

**Checklist**:
- Ensure user is properly initialized: `UserManager.shared.isValidUser`
- Check network connectivity
- Verify event type and parameters
- Enable debug logging to see detailed information

#### 4. App Store Submission Issues

**Problem**: App rejected due to tracking permissions

**Solution**: Add tracking usage description to `Info.plist`:
```xml
<key>NSUserTrackingUsageDescription</key>
<string>This app uses advertising identifiers to provide personalized content and improve user experience.</string>
```

### Testing

#### Reset User Data (Debug Only)

```swift
#if DEBUG
// Clear all user data for testing
UserManager.shared.clearUserData()

// Reinitialize user
UserManager.shared.initializeUser()
#endif
```

#### Manual Event Testing

```swift
// Test different event types
UserManager.shared.createSession(eventType: .appLaunch, vertical: .general)
UserManager.shared.createSession(eventType: .searchButtonClick, vertical: .flight)
UserManager.shared.createSession(eventType: .resultSelected, vertical: .flight)
```

## 📄 License

This tracking system is designed for the D2Flight iOS application. Modify as needed for your specific use case.

## 🤝 Contributing

When contributing to this system:

1. Maintain backward compatibility
2. Add appropriate logging for new features
3. Update this README with any new functionality
4. Test thoroughly with both IDFA available and unavailable scenarios
5. Ensure proper error handling for network failures
