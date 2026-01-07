# melonJS Spine Plugin

A powerful [Spine](http://en.esotericsoftware.com/spine-in-depth) 4.x skeletal animation plugin for [melonJS 2](http://www.melonjs.org). Bring your characters to life with smooth, professional 2D skeletal animations!

![melonjs-spine-gif](https://github.com/melonjs/spine-plugin/assets/4033090/dc259c8e-def6-419e-83a9-cda374715686)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/melonjs/es6-boilerplate/blob/master/LICENSE)
[![NPM Package](https://img.shields.io/npm/v/@melonjs/spine-plugin)](https://www.npmjs.com/package/@melonjs/spine-plugin)
[![jsDeliver](https://data.jsdelivr.com/v1/package/npm/@melonjs/spine-plugin/badge?style=rounded)](https://www.jsdelivr.com/package/npm/@melonjs/spine-plugin)

> **Note:** Although functional, this plugin is still a work in progress. Feedback and contributions are welcome!

---

## 📋 Table of Contents

- [Features](#-features)
- [Installation](#-installation)
- [Quick Start](#-quick-start)
- [Basic Usage](#-basic-usage)
- [Advanced Examples](#-advanced-examples)
- [API Reference](#-api-reference)
- [Compatibility](#-compatibility)
- [Support](#-questions-need-help)

---

## ✨ Features

- 🎭 Full Spine 4.x runtime support (4.1 and 4.2-beta)
- 🎨 WebGL and Canvas rendering
- 🔄 Smooth animation blending and transitions
- 👥 Multiple skeleton support per object
- 🎯 Event callbacks for animation lifecycle
- 🖼️ Skin and attachment management
- 🐛 Built-in debug rendering
- ⚡ Optimized performance
- 📦 Bundled with Spine runtime - no separate installation needed

---

## 📦 Installation

### Requirements

- melonJS version **15.12** or higher
- Node.js and npm

### Install via npm

```bash
npm install @melonjs/spine-plugin
```

### Install via yarn

```bash
yarn add @melonjs/spine-plugin
```

---

## 🚀 Quick Start

### 1. Register the Plugin

```javascript
import { SpinePlugin } from '@melonjs/spine-plugin';
import * as me from 'melonjs';

// Register the plugin with melonJS
me.plugin.register(SpinePlugin);
```

### 2. Prepare Your Assets

```javascript
const DataManifest = [
    {
        name: "character.json",
        type: "spine",  // Important: use "spine" type
        src: "data/spine/character.json"
    },
    {
        name: "character.atlas",
        type: "spine",
        src: "data/spine/character.atlas"
    }
];
```

### 3. Create and Use Spine Animations

```javascript
import Spine from '@melonjs/spine-plugin';

me.loader.preload(DataManifest, async function() {
    // Create a Spine renderable
    let character = new Spine(100, 100, {
        width: 200,
        height: 300,
        atlasFile: "character.atlas",
        jsonFile: "character.json"
    });

    // Play an animation
    character.setAnimation(0, "idle", true);

    // Add to game world
    me.game.world.addChild(character);
});
```

---

## 📖 Basic Usage

### Creating a Basic Character

```javascript
import Spine from '@melonjs/spine-plugin';

class Player extends me.Entity {
    constructor(x, y) {
        super(x, y, {width: 64, height: 64});
        
        // Create spine animation
        this.spineCharacter = new Spine(0, 0, {
            width: 200,
            height: 300,
            atlasFile: "hero.atlas",
            jsonFile: "hero.json",
            skinName: "default",
            mixTime: 0.2  // Smooth transition time
        });
        
        // Start with idle animation
        this.spineCharacter.setAnimation(0, "idle", true);
        
        this.body.addShape(new me.Rect(0, 0, 64, 64));
    }
    
    update(dt) {
        // Update spine animation
        this.spineCharacter.update(dt);
        return super.update(dt);
    }
    
    draw(renderer) {
        // Draw spine character
        this.spineCharacter.draw(renderer);
    }
}
```

### Playing Animations

```javascript
// Play animation once
character.setAnimation(0, "attack", null, false);

// Play animation with loop
character.setAnimation(0, "walk", null, true);

// Play with callback when complete
character.setAnimation(0, "jump", () => {
    console.log("Jump completed!");
    character.setAnimation(0, "idle", null, true);
}, false);
```

### Changing Character Direction

```javascript
// Flip horizontally (face left/right)
character.flipX(true);  // Face left
character.flipX(false); // Face right

// Flip vertically
character.flipY(true);
```

### Changing Skins

```javascript
// Change character appearance
character.setSkinByName("summer_outfit");
character.setSkinByName("winter_outfit");
character.setSkinByName("armor");
```

---

## 🎯 Advanced Examples

### Character with State-Based Animations

```javascript
class Hero extends me.Entity {
    constructor(x, y) {
        super(x, y, {width: 64, height: 64});
        
        this.spine = new Spine(0, 0, {
            width: 200,
            height: 300,
            atlasFile: "hero.atlas",
            jsonFile: "hero.json",
            mixTime: 0.15
        });
        
        this.state = "idle";
        this.spine.setAnimation(0, "idle", true);
    }
    
    update(dt) {
        let newState = this.state;
        
        // Determine state based on input
        if (me.input.isKeyPressed("left")) {
            newState = "run";
            this.spine.flipX(true);
            this.pos.x -= 2;
        } 
        else if (me.input.isKeyPressed("right")) {
            newState = "run";
            this.spine.flipX(false);
            this.pos.x += 2;
        }
        else if (me.input.isKeyPressed("space")) {
            newState = "attack";
        }
        else {
            newState = "idle";
        }
        
        // Update animation if state changed
        if (newState !== this.state) {
            this.state = newState;
            
            if (newState === "attack") {
                // Attack once, then return to idle
                this.spine.setAnimation(0, "attack", () => {
                    this.state = "idle";
                    this.spine.setAnimation(0, "idle", null, true);
                }, false);
            } else {
                this.spine.setAnimation(0, newState, null, true);
            }
        }
        
        this.spine.pos.set(this.pos.x, this.pos.y);
        this.spine.update(dt);
        return super.update(dt);
    }
    
    draw(renderer) {
        this.spine.draw(renderer);
    }
}
```

### Animation Sequencing

```javascript
// Play a sequence of animations
character.setAnimation(0, "run", () => {
    character.setAnimation(0, "jump", () => {
        character.setAnimation(0, "land", () => {
            character.setAnimation(0, "idle", null, true);
        }, false);
    }, false);
}, false);

// Or use animation queuing
character.setAnimation(0, "run", null, false);
character.addAnimationByName(0, "jump", false, 0);
character.addAnimationByName(0, "land", false, 0);
character.addAnimationByName(0, "idle", true, 0);
```

### Multiple Skeletons (Character Customization)

```javascript
// Create character with multiple skeleton parts
let character = new Spine(100, 100, {
    width: 200,
    height: 300,
    skeletonConfigs: [
        { 
            atlasFile: "body.atlas", 
            jsonFile: "body.json", 
            skin: "default" 
        },
        { 
            atlasFile: "armor.atlas", 
            jsonFile: "armor.json", 
            skin: "plate" 
        },
        { 
            atlasFile: "weapon.atlas", 
            jsonFile: "weapon.json", 
            skin: "sword" 
        }
    ]
});

// Switch between skeleton sets
character.setSkeletonToDraw(0); // Show body
character.setSkeletonToDraw(1); // Show armor
character.setSkeletonToDraw(2); // Show weapon

// Or cycle through
character.nextSkeleton();
character.previousSkeleton();

// Animate specific skeleton
character.setAnimationForSkeleton(1, 0, "glow", null, true);
```

### Smooth Animation Transitions

```javascript
// Set default blend time for all transitions
character.setDefaultMixTime(0.3);

// Set specific transition times
character.setTransitionMixTime("walk", "run", 0.1);
character.setTransitionMixTime("run", "jump", 0.2);
character.setTransitionMixTime("idle", "attack", 0.05);
```

### Interactive NPC

```javascript
class NPC extends me.Entity {
    constructor(x, y) {
        super(x, y, {width: 64, height: 64});
        
        this.spine = new Spine(0, 0, {
            width: 150,
            height: 200,
            atlasFile: "npc.atlas",
            jsonFile: "npc.json"
        });
        
        this.spine.setAnimation(0, "idle", null, true);
        this.isInteracting = false;
    }
    
    onPlayerNear() {
        if (!this.isInteracting) {
            // Wave at player
            this.spine.setAnimation(0, "wave", () => {
                this.spine.setAnimation(0, "idle", null, true);
            }, false);
        }
    }
    
    onInteract() {
        this.isInteracting = true;
        
        // Talk animation
        this.spine.setAnimation(0, "talk", null, true);
        
        // Show dialog...
        // When dialog closes:
        this.onDialogClose = () => {
            this.spine.setAnimation(0, "idle", null, true);
            this.isInteracting = false;
        };
    }
    
    update(dt) {
        this.spine.pos.set(this.pos.x, this.pos.y);
        this.spine.update(dt);
        return super.update(dt);
    }
    
    draw(renderer) {
        this.spine.draw(renderer);
    }
}
```

### Debug Rendering

```javascript
// Enable debug mode to see bones and boundaries
character.debugRendering = true;

// Useful during development
if (DEBUG_MODE) {
    character.debugRendering = true;
}
```

### Scale and Transform

```javascript
// Scale character
character.scale(1.5);      // 150% size
character.scale(0.8, 1.2); // Stretch effect

// Rotate
character.rotate(Math.PI / 4); // 45 degrees

// Position
let pos = character.getSpinePosition();
console.log(pos.x, pos.y);

// Resize
character.setSpineSize(300, 400);
```

### Working with Tracks

```javascript
// Get current animation info
let current = character.getCurrentAnimation(0);
if (current) {
    console.log(`Playing: ${current.name}`);
    console.log(`Looping: ${current.loop}`);
}

// Check specific animation
if (character.isCurrentAnimation("attack")) {
    console.log("Character is attacking!");
}

// Multiple tracks for layered animations
character.setAnimation(0, "walk", null, true);  // Legs
character.setAnimation(1, "shoot", null, false); // Upper body
```

### Memory Management

```javascript
class GameScene extends me.Stage {
    onDestroyEvent() {
        // Always dispose spine objects when done
        if (this.character && this.character.spine) {
            this.character.spine.dispose();
        }
        
        // Clean up all spine objects
        this.spineCharacters.forEach(char => {
            char.dispose();
        });
    }
}
```

---

## 📚 API Reference

### Constructor

```javascript
new Spine(x, y, settings)
```

**Parameters:**
- `x` (number) - X position
- `y` (number) - Y position
- `settings` (Object):
  - `width` (number) - Width
  - `height` (number) - Height
  - `atlasFile` (string) - Atlas file name
  - `jsonFile` (string) - JSON skeleton file
  - `skinName` (string, optional) - Default skin
  - `mixTime` (number, optional) - Blend time (default: 0.2)
  - `z` (number, optional) - Z-index
  - `skeletonConfigs` (Array, optional) - Multiple skeletons

### Key Properties

| Property | Type | Description |
|----------|------|-------------|
| `skeleton` | Skeleton | Current active skeleton (read-only) |
| `animationState` | AnimationState | Current animation state (read-only) |
| `currentTrack` | TrackEntry | Current animation track |
| `debugRendering` | boolean | Enable/disable debug mode |

### Animation Methods

| Method | Description |
|--------|-------------|
| `setAnimation(track, name, callback, loop)` | Play animation by name |
| `setAnimationByIndex(track, index, callback, loop)` | Play animation by index |
| `addAnimationByName(track, name, loop, delay)` | Queue animation |
| `getCurrentAnimation(track)` | Get current animation info |
| `isCurrentAnimation(name)` | Check if animation is playing |

### Skeleton Management

| Method | Description |
|--------|-------------|
| `addSkeleton(atlas, json, skin, index)` | Add new skeleton |
| `removeSkeleton(index)` | Remove skeleton |
| `setSkeletonToDraw(index)` | Set active skeleton |
| `nextSkeleton()` | Switch to next skeleton |
| `previousSkeleton()` | Switch to previous skeleton |
| `getSkeletonCount()` | Get total skeletons |

### Transformation

| Method | Description |
|--------|-------------|
| `flipX(flip)` | Flip horizontally |
| `flipY(flip)` | Flip vertically |
| `scale(x, y)` | Scale the spine object |
| `rotate(angle, point)` | Rotate the spine object |

### Skin & Appearance

| Method | Description |
|--------|-------------|
| `setSkinByName(name)` | Change skin |
| `createSkin(name)` | Create new skin |
| `setToSetupPose()` | Reset to default pose |

### Mixing & Transitions

| Method | Description |
|--------|-------------|
| `setDefaultMixTime(time)` | Set default blend time |
| `setTransitionMixTime(from, to, time)` | Set specific transition |

### Utility

| Method | Description |
|--------|-------------|
| `dispose()` | Clean up resources |
| `getSpinePosition()` | Get position as Vector2d |
| `setSpineSize(w, h)` | Set dimensions |
| `getSpineSize()` | Get dimensions |

For complete API documentation, see [Full API Reference](API.md).

---

## 🔧 Compatibility

| melonJS | @melonjs/spine-plugin | spine-runtime |
|---------|----------------------|---------------|
| v15.12.x or higher | v1.x | v4.1, v4.2-beta |

> **Note:** The current plugin is bundled with Spine runtime 4.2.x beta, which maintains backward compatibility with Spine 4.1 exports.

---

## 💡 Tips & Best Practices

### Performance

1. **Reuse Spine objects** instead of creating new ones
2. **Dispose** objects when no longer needed
3. **Disable debug rendering** in production
4. Use **appropriate mix times** - shorter for fast actions, longer for smooth transitions

### Animation Design

1. **Use callbacks** for animation chaining instead of polling
2. **Set appropriate loop** values - true for idle/walk, false for attacks
3. **Layer animations** using multiple tracks for complex behavior
4. **Pre-configure transitions** for commonly used animation pairs

### Common Patterns

```javascript
// Idle → Action → Idle pattern
character.setAnimation(0, "action", () => {
    character.setAnimation(0, "idle", null, true);
}, false);

// State machine pattern
const STATES = {
    IDLE: "idle",
    WALK: "walk",
    ATTACK: "attack"
};

updateState(newState) {
    if (this.currentState !== newState) {
        this.currentState = newState;
        this.spine.setAnimation(0, newState, null, 
            newState !== STATES.ATTACK);
    }
}
```

---

## 🐛 Troubleshooting

### Animation not playing

```javascript
// Check if animation exists
let animations = spine.skeleton.data.animations;
console.log("Available animations:", animations.map(a => a.name));

// Verify animation state
let current = spine.getCurrentAnimation(0);
console.log("Current animation:", current);
```

### Character appears flipped

```javascript
// Reset flipping
spine.flipX(false);
spine.flipY(false);

// Or reset to setup pose
spine.setToSetupPose();
```

### Memory leaks

```javascript
// Always dispose when removing
entity.spine.dispose();
entity = null;
```

---

## 📝 Examples Repository

Check out the [test folder](test) for complete working examples including:
- Basic character animation
- State machines
- Multiple skeletons
- Interactive NPCs
- Animation sequencing

---

## ❓ Questions, Need Help?

### Community Support

- **Forums:** [melonJS Discourse](https://melonjs.discourse.group) | [HTML5 Game Devs](http://www.html5gamedevs.com/forum/32-melonjs/)
- **Chat:** [Discord](https://discord.gg/aur7JMk) | [Gitter](https://gitter.im/melonjs/public)
- **Wiki:** [melonJS Wiki](https://github.com/melonjs/melonJS/wiki)

### Reporting Issues

Found a bug? Please [open an issue](https://github.com/melonjs/spine-plugin/issues) with:
- melonJS version
- Spine plugin version
- Spine runtime version used to export
- Steps to reproduce
- Code sample if possible

---

## 🤝 Contributing

Contributions are welcome! Feel free to:
- Report bugs
- Suggest features
- Submit pull requests
- Improve documentation

---

## 📄 License

This project is licensed under the MIT License.

---

## 🙏 Acknowledgments

- [Esoteric Software](http://esotericsoftware.com/) for Spine
- [melonJS](http://melonjs.org) team and community
- All contributors and users

---

Made with ❤️ for the melonJS community