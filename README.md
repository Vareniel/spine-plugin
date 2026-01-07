# melonJS Spine Plugin

A [Spine](http://en.esotericsoftware.com/spine-in-depth) 4.x skeletal animation plugin for [melonJS 2](http://www.melonjs.org).

![melonjs-spine-gif](https://github.com/melonjs/spine-plugin/assets/4033090/dc259c8e-def6-419e-83a9-cda374715686)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/melonjs/es6-boilerplate/blob/master/LICENSE)
[![NPM Package](https://img.shields.io/npm/v/@melonjs/spine-plugin)](https://www.npmjs.com/package/@melonjs/spine-plugin)
[![jsDeliver](https://data.jsdelivr.com/v1/package/npm/@melonjs/spine-plugin/badge?style=rounded)](https://www.jsdelivr.com/package/npm/@melonjs/spine-plugin)

> **Note:** This plugin is bundled with Spine 4.x runtime - no separate installation needed. Requires melonJS v15.12 or higher.

---

## Installation

```bash
npm install @melonjs/spine-plugin
```

---

## Quick Start

### 1. Register Plugin

```javascript
import { SpinePlugin } from '@melonjs/spine-plugin';
import * as me from 'melonjs';

me.plugin.register(SpinePlugin);
```

### 2. Load Assets

```javascript
const resources = [
    { name: "character.json", type: "spine", src: "data/character.json" },
    { name: "character.atlas", type: "spine", src: "data/character.atlas" }
];

me.loader.preload(resources, () => {
    // Assets loaded, start game
});
```

### 3. Create Spine Object

**Spine extends Renderable** - use it directly or extend it:

```javascript
import Spine from '@melonjs/spine-plugin';

// Option A: Direct usage
let character = new Spine(100, 100, {
    width: 200,
    height: 300,
    atlasFile: "character.atlas",
    jsonFile: "character.json"
});

character.setAnimation(0, "idle", null, true);
me.game.world.addChild(character);

// Option B: Extend for custom behavior (Recommended)
export default class Player extends Spine {
    constructor(x, y) {
        super(x, y, {
            width: 200,
            height: 300,
            atlasFile: "hero.atlas",
            jsonFile: "hero.json"
        });
        
        this.body.addShape(new me.Rect(0, 0, 64, 64));
        this.setAnimation(0, "idle", null, true);
    }
    
    update(dt) {
        super.update(dt); // Important: call parent
        this.body.update(dt);
        return true;
    }
    
    draw(renderer) {
        super.draw(renderer); // Important: call parent
    }
}
```

---

## Basic Usage

### Playing Animations

```javascript
// Play animation (track, name, callback, loop)
character.setAnimation(0, "walk", null, true);

// Play with callback
character.setAnimation(0, "attack", () => {
    console.log("Attack finished!");
}, false);

// Play by index
character.setAnimationByIndex(0, 2, null, true);
```

### Flipping Character

```javascript
// Flip horizontally
character.flipX(true);  // Face left
character.flipX(false); // Face right

// Flip vertically
character.flipY(true);
```

### Changing Skins

```javascript
character.setSkinByName("armor");
character.setSkinByName("summer_outfit");
```

### Animation Blending

```javascript
// Set global blend time
character.setDefaultMixTime(0.2);

// Set specific transitions
character.setTransitionMixTime("walk", "run", 0.1);
character.setTransitionMixTime("idle", "attack", 0.05);
```

### Checking Current Animation

```javascript
// Get animation info
let current = character.getCurrentAnimation(0);
if (current) {
    console.log(current.name);  // "walk"
    console.log(current.loop);  // true
}

// Check specific animation
if (character.isCurrentAnimation("idle")) {
    // Do something
}
```

---

## API Reference

### Constructor

```javascript
new Spine(x, y, settings)
```

**Settings Object:**
```javascript
{
    width: 200,              // Width
    height: 300,             // Height
    atlasFile: "char.atlas", // Atlas file
    jsonFile: "char.json",   // Skeleton JSON
    skinName: "default",     // Initial skin (optional)
    mixTime: 0.2,            // Blend time (optional)
    anchorPoint: {x: 0.5, y: 1} // Pivot point (optional)
}
```

### Animation Methods

#### `setAnimation(trackIndex, name, callback, loop)`
Play an animation on specified track.

**Parameters:**
- `trackIndex` (number) - Track index (usually 0)
- `name` (string) - Animation name
- `callback` (function) - Called when animation completes (optional)
- `loop` (boolean) - Loop animation (default: false)

```javascript
character.setAnimation(0, "run", null, true);
```

#### `setAnimationByIndex(trackIndex, index, callback, loop)`
Play animation by its index in the animation list.

```javascript
character.setAnimationByIndex(0, 2, null, false);
```

#### `addAnimationByName(trackIndex, name, loop, delay)`
Queue animation after current one.

```javascript
character.addAnimationByName(0, "jump", false, 0.5);
```

#### `getCurrentAnimation(trackIndex)`
Get current animation info.

**Returns:** `{name, trackIndex, loop, track}` or `null`

```javascript
let anim = character.getCurrentAnimation(0);
console.log(anim.name); // "walk"
```

#### `isCurrentAnimation(name)`
Check if specific animation is playing.

```javascript
if (character.isCurrentAnimation("attack")) {
    // Character is attacking
}
```

### Skin Methods

#### `setSkinByName(skinName)`
Change character skin.

```javascript
character.setSkinByName("winter_outfit");
```

#### `createSkin(name)`
Create new empty skin.

```javascript
let customSkin = character.createSkin("custom");
```

### Transform Methods

#### `flipX(flip)`
Flip horizontally.

```javascript
character.flipX(true);  // Mirror horizontally
```

#### `flipY(flip)`
Flip vertically.

```javascript
character.flipY(true);
```

#### `scale(x, y)`
Scale the spine object.

```javascript
character.scale(1.5);      // Scale uniformly
character.scale(1.2, 0.8); // Scale X and Y separately
```

#### `rotate(angle, point)`
Rotate the spine object.

```javascript
character.rotate(Math.PI / 4); // Rotate 45 degrees
```

### Mixing Methods

#### `setDefaultMixTime(mixTime)`
Set default blend time for all animations.

```javascript
character.setDefaultMixTime(0.3);
```

#### `setTransitionMixTime(fromAnimation, toAnimation, mixTime)`
Set blend time between specific animations.

```javascript
character.setTransitionMixTime("walk", "run", 0.1);
```

### Multiple Skeleton Methods

#### `addSkeleton(atlasFile, jsonFile, skinName, index)`
Add new skeleton to collection.

```javascript
let index = character.addSkeleton("weapon.atlas", "weapon.json", "sword");
```

#### `setSkeletonToDraw(index)`
Set which skeleton to render.

```javascript
character.setSkeletonToDraw(0);
```

#### `nextSkeleton()` / `previousSkeleton()`
Switch between skeletons.

```javascript
character.nextSkeleton();
character.previousSkeleton();
```

#### `getSkeletonCount()`
Get total number of skeletons.

```javascript
let count = character.getSkeletonCount();
```

#### `setAnimationForSkeleton(skeletonIndex, trackIndex, name, callback, loop)`
Play animation on specific skeleton.

```javascript
character.setAnimationForSkeleton(1, 0, "glow", null, true);
```

### Utility Methods

#### `getSpinePosition()`
Get position as Vector2d.

```javascript
let pos = character.getSpinePosition();
console.log(pos.x, pos.y);
```

#### `setSpineSize(width, height)`
Set dimensions.

```javascript
character.setSpineSize(250, 350);
```

#### `getSpineSize()`
Get dimensions.

```javascript
let size = character.getSpineSize();
console.log(size.width, size.height);
```

#### `setToSetupPose()`
Reset skeleton to initial pose.

```javascript
character.setToSetupPose();
```

#### `dispose()`
Clean up resources (call when removing).

```javascript
character.dispose();
```

### Properties

#### `skeleton` (read-only)
Current active skeleton object.

#### `animationState` (read-only)
Current animation state object.

#### `currentTrack`
Current animation track entry.

#### `debugRendering`
Enable/disable debug visualization.

```javascript
character.debugRendering = true; // Show bones
```

---

## Common Patterns

### Simple Character Movement

```javascript
export default class Hero extends Spine {
    constructor(x, y) {
        super(x, y, {
            width: 200,
            height: 300,
            atlasFile: "hero.atlas",
            jsonFile: "hero.json"
        });
        
        this.body.addShape(new me.Rect(0, 0, 64, 64));
        this.state = "idle";
        this.setAnimation(0, "idle", null, true);
    }
    
    update(dt) {
        super.update(dt);
        
        let newState = "idle";
        
        if (me.input.isKeyPressed("left")) {
            this.body.force.x = -3;
            this.flipX(true);
            newState = "walk";
        } else if (me.input.isKeyPressed("right")) {
            this.body.force.x = 3;
            this.flipX(false);
            newState = "walk";
        }
        
        if (newState !== this.state) {
            this.state = newState;
            this.setAnimation(0, newState, null, true);
        }
        
        this.body.update(dt);
        return true;
    }
    
    draw(renderer) {
        super.draw(renderer);
    }
}
```

### Attack Animation

```javascript
attack() {
    this.setAnimation(0, "attack", () => {
        // Attack finished, return to idle
        this.setAnimation(0, "idle", null, true);
    }, false);
}
```

### Animation Sequence

```javascript
// Play sequence: run → jump → land → idle
character.setAnimation(0, "run", () => {
    character.setAnimation(0, "jump", () => {
        character.setAnimation(0, "land", () => {
            character.setAnimation(0, "idle", null, true);
        }, false);
    }, false);
}, false);
```

### State Machine

```javascript
updateAnimation() {
    let newState = "idle";
    
    if (this.body.vel.y < 0) {
        newState = "jump";
    } else if (this.body.vel.y > 0) {
        newState = "fall";
    } else if (this.body.vel.x !== 0) {
        newState = "run";
    }
    
    if (newState !== this.state) {
        this.state = newState;
        this.setAnimation(0, newState, null, true);
    }
}
```

---

## Best Practices

### ✅ DO

```javascript
// Extend Spine directly
class Hero extends Spine { ... }

// Always call super.update() and super.draw()
update(dt) {
    super.update(dt);
    // your code
    return true;
}

// Dispose when done
onDeactivateEvent() {
    this.dispose();
}

// Use callbacks for sequences
this.setAnimation(0, "attack", () => {
    this.setAnimation(0, "idle", null, true);
}, false);
```

### ❌ DON'T

```javascript
// Don't extend Entity/Sprite
class Hero extends me.Entity { ... } // Wrong!

// Don't forget super calls
update(dt) {
    // your code
    return true; // Missing super.update(dt)!
}

// Don't leave debug mode on
character.debugRendering = true; // Remove in production
```

---

## Troubleshooting

### Animation not playing

```javascript
// Check available animations
console.log(character.skeleton.data.animations.map(a => a.name));

// Check current animation
let current = character.getCurrentAnimation(0);
console.log("Current:", current?.name);
```

### Character not visible

```javascript
// Enable debug mode
character.debugRendering = true;

// Check dimensions
console.log(character.width, character.height);

// Check position
console.log(character.pos.x, character.pos.y);
```

### Wrong direction

```javascript
// Reset flipping
character.flipX(false);
character.flipY(false);
```

---

## Compatibility

| melonJS | @melonjs/spine-plugin | spine-runtime |
|---------|----------------------|---------------|
| v15.12+ | v1.x | v4.1, v4.2-beta |

> The plugin is bundled with Spine 4.2-beta runtime, backward compatible with Spine 4.1 exports.

---

## Support

- **Forums:** [melonJS Discourse](https://melonjs.discourse.group)
- **Chat:** [Discord](https://discord.gg/aur7JMk) | [Gitter](https://gitter.im/melonjs/public)
- **Wiki:** [melonJS Wiki](https://github.com/melonjs/melonJS/wiki)
- **Issues:** [GitHub Issues](https://github.com/melonjs/spine-plugin/issues)

---

## License

MIT License

---

Made with ❤️ for the melonJS community