# Polymorph

[![License][license-image]][license-url]
![swift](https://img.shields.io/badge/Swift-5.10%20|%205.9-orange)
![visionOS](https://img.shields.io/badge/visionOS-blue)
![iOS](https://img.shields.io/badge/iOS-blue)

A DSL for vending SwiftUI views that allows the user to change attributes on a RealityKit `Entity`. Enabling a type-safe way of reaching out to RealityKit content with a readable syntax. 

![Simulator Screen Recording - Apple Vision Pro - 2024-03-28 at 15 09 05](https://github.com/loomery/RealityShaper/assets/59975039/bf2a2788-ae9c-4254-88a5-90177087b0b6)

## Installation

### SPM

Add this project on your `Package.swift`

```swift
import PackageDescription

let package = Package(
    dependencies: [
        .Package(url: "https://github.com/loomery/Polymorph.git", majorVersion: 0, minor: 0)
    ]
)
```

## Usage example

Conform your Entity model to the `EntityRepresentable` protocol and setup the `init` as follows:

```swift
import Polymorph

struct ShowroomCar: EntityRepresentable {
    let name: String
    var entity: RealityKit.Entity
    var subEntities: [Library.Entity]
    
    init(named name: String, @EntityBuilder subEntities: () async -> [Library.Entity]) async throws {
        self.name = name
        let entity = try await RealityKit.Entity(named: name, in: realityKitContentBundle)
        
        self.subEntities = await subEntities().map {
            $0.withEntity(entity)
        }
        self.entity = entity
    }
}
```

The package comes with a `resultBuilder` called `EntityBuilder` that allows you to define the sub entities that are bound to the materials in Reality Composer Pro:

<img width="302" alt="Screenshot 2024-03-28 at 16 05 20" src="https://github.com/loomery/Polymorph/assets/59975039/a86e53aa-fb9d-41a9-ac7d-790ee8a531e6">

```swift
extension ShowroomCar {
    static var p1: () async -> Self = {
        try await ShopItem(named: "P1") {
            Library.Entity(named: "Paintjob") {
                Library.Color(name: "Paintjob")
            }
            Library.Entity(named: "Interior") {
                Library.Color(name: "Interior")
            }
            Library.Entity(named: "Seats") {
                Library.Color(name: "Seats")
            }
            Library.Entity(named: "Rims") {
                Library.Color(name: "Rims")
            }
        }
    }
}
```

Then you can load in the entity into the immersive space:

```swift
RealityView {
    subscriptions.append(content.subscribe(
        to: ComponentEvents.DidAdd.self,
        componentType: CustomizableItemComponent.self
    ) { event in
        let car = await ShowroomCar.p1()
        await event.entity.children.append(car.entity)
    })
} ...
```

And add the SwiftUI views to the window:

```swift
ScrollView {
    VStack {
        Text("Customise your item.")
        ForEach(showroomCar.attributes) { attribute in
            attribute
        }
    }
}
```

<img width="536" alt="Screenshot 2024-03-28 at 16 07 52" src="https://github.com/loomery/Polymorph/assets/59975039/816b2baa-06bd-4e1a-8a2b-9440e20e0b26">

## Release History

* 0.0.1
    * Work in progress

## Meta

Tom Holmes – [@tommy_holmes_](https://twitter.com/tommy_holmes_) – tom@loomery.com

Distributed under the MIT license. See ``LICENSE`` for more information.

[Tom Holmes GitHub](https://github.com/tommy-holmes/) 
