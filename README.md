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
- [Usage Patterns](#-usage-patterns)
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
- 🎮 **Already extends Renderable** - no need to wrap in Entity or Sprite!

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

Since **Spine already extends Renderable**, you can use it directly or extend it for custom behavior:

```javascript
import Spine from '@melonjs/spine-plugin';

me.loader.preload(DataManifest, async function() {
    // Option 1: Use directly
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

## 🎯 Usage Patterns

### Pattern 1: Direct Usage (Simple)

Use Spine directly when you don't need custom logic:

```javascript
import Spine from '@melonjs/spine-plugin';

// Create spine object
let enemy = new Spine(200, 150, {
    width: 150,
    height: 200,
    atlasFile: "enemy.atlas",
    jsonFile: "enemy.json",
    skinName: "default"
});

// Set animation
enemy.setAnimation(0, "walk", null, true);

// Add collision body if needed
enemy.body.addShape(new me.Rect(0, 0, 64, 64));

// Add to world
me.game.world.addChild(enemy);
```

### Pattern 2: Extend Spine (Recommended)

Extend Spine class for characters with custom logic:

```javascript
import Spine from '@melonjs/spine-plugin';

export default class Player extends Spine {
    constructor(x, y, settings = {}) {
        // Set default dimensions
        settings.width = settings.width || 200;
        settings.height = settings.height || 300;
        
        // Configure spine files
        settings.jsonFile = "hero.json";
        settings.atlasFile = "hero.atlas";
        settings.skinName = "default";
        settings.mixTime = 0.2;
        
        // Set anchor point
        settings.anchorPoint = {
            x: 0.5,
            y: 1
        };
        
        // Call parent constructor
        super(x, y, settings);
        
        // Add collision body
        this.body.addShape(new me.Rect(0, 0, 64, 64));
        
        // Set velocity
        this.body.setMaxVelocity(3, 15);
        this.body.setFriction(0.4, 0);
        
        // Custom properties
        this.state = "idle";
        this.health = 100;
        
        // Start with idle animation
        this.setAnimation(0, "idle", null, true);
    }
    
    update(dt) {
        // Call parent update (handles spine animation)
        super.update(dt);
        
        // Your custom logic here
        this.handleInput();
        this.updateAnimation();
        
        // Apply physics
        this.body.update(dt);
        
        return true;
    }
    
    handleInput() {
        if (me.input.isKeyPressed("left")) {
            this.body.force.x = -this.body.maxVel.x;
            this.flipX(true);
        } else if (me.input.isKeyPressed("right")) {
            this.body.force.x = this.body.maxVel.x;
            this.flipX(false);
        }
        
        if (me.input.isKeyPressed("jump")) {
            if (!this.body.jumping && !this.body.falling) {
                this.body.force.y = -this.body.maxVel.y;
            }
        }
    }
    
    updateAnimation() {
        let newState = "idle";
        
        if (this.body.vel.y !== 0) {
            newState = this.body.vel.y < 0 ? "jump" : "fall";
        } else if (this.body.vel.x !== 0) {
            newState = "run";
        }
        
        if (newState !== this.state) {
            this.state = newState;
            this.setAnimation(0, newState, null, true);
        }
    }
    
    takeDamage(amount) {
        this.health -= amount;
        this.setAnimation(0, "hit", () => {
            if (this.health > 0) {
                this.setAnimation(0, "idle", null, true);
            } else {
                this.die();
            }
        }, false);
    }
    
    die() {
        this.setAnimation(0, "death", () => {
            me.game.world.removeChild(this);
        }, false);
    }
    
    draw(renderer) {
        // Call parent draw (handles spine rendering)
        super.draw(renderer);
    }
}
```

### Pattern 3: Complete Game Character Example

```javascript
import Spine from '@melonjs/spine-plugin';

export default class PlayerSpine extends Spine {
    constructor(x, y, settings = {}) {
        // Configure default settings
        settings.width = settings.width || 401;
        settings.height = settings.height || 377.5;
        settings.anchorPoint = {
            x: 0,
            y: 0
        };
        
        settings.jsonFile = "Spine_Man.json";
        settings.atlasFile = "Spine_Man.atlas";
        settings.mixTime = 0.1;
        
        // Call Spine constructor
        super(x, y, settings);
        
        // Add physics body
        this.body.addShape(new me.Rect(0, 0, 64, 64));
        this.body.setMaxVelocity(5, 15);
        this.body.setFriction(0.5, 0);
        
        // Game properties
        this.state = "idle";
        this.isAttacking = false;
        this.canAttack = true;
        
        // Start animation
        this.setAnimation(0, "idle", null, true);
    }
    
    update(dt) {
        // Update spine animation system
        let drawNextFrame = super.update(dt);
        
        // Handle input
        if (!this.isAttacking) {
            this.handleMovement();
            this.handleActions();
        }
        
        // Update physics
        this.body.update(dt);
        
        return drawNextFrame;
    }
    
    handleMovement() {
        if (me.input.isKeyPressed("left")) {
            this.body.force.x = -this.body.maxVel.x;
            this.flipX(true);
            if (this.state !== "walk") {
                this.state = "walk";
                this.setAnimation(0, "walk", null, true);
            }
        } else if (me.input.isKeyPressed("right")) {
            this.body.force.x = this.body.maxVel.x;
            this.flipX(false);
            if (this.state !== "walk") {
                this.state = "walk";
                this.setAnimation(0, "walk", null, true);
            }
        } else {
            if (this.state !== "idle") {
                this.state = "idle";
                this.setAnimation(0, "idle", null, true);
            }
        }
    }
    
    handleActions() {
        if (me.input.isKeyPressed("attack") && this.canAttack) {
            this.attack();
        }
        
        if (me.input.isKeyPressed("jump") && !this.body.jumping) {
            this.jump();
        }
    }
    
    attack() {
        this.isAttacking = true;
        this.canAttack = false;
        this.state = "attack";
        
        this.setAnimation(0, "attack", () => {
            this.isAttacking = false;
            this.state = "idle";
            this.setAnimation(0, "idle", null, true);
            
            // Cooldown
            setTimeout(() => {
                this.canAttack = true;
            }, 500);
        }, false);
    }
    
    jump() {
        this.body.force.y = -this.body.maxVel.y;
        this.state = "jump";
        this.setAnimation(0, "jump", () => {
            this.state = "idle";
            this.setAnimation(0, "idle", null, true);
        }, false);
    }
    
    draw(renderer) {
        super.draw(renderer);
    }
    
    onActivateEvent() {
        // Register input when entity is added to world
        me.input.bindKey(me.input.KEY.LEFT, "left");
        me.input.bindKey(me.input.KEY.RIGHT, "right");
        me.input.bindKey(me.input.KEY.UP, "jump");
        me.input.bindKey(me.input.KEY.SPACE, "attack");
    }
    
    onDeactivateEvent() {
        // Unregister input when entity is removed
        me.input.unbindKey(me.input.KEY.LEFT);
        me.input.unbindKey(me.input.KEY.RIGHT);
        me.input.unbindKey(me.input.KEY.UP);
        me.input.unbindKey(me.input.KEY.SPACE);
    }
}
```

---

## 🎮 Advanced Examples

### NPC with Dialog System

```javascript
import Spine from '@melonjs/spine-plugin';

export default class NPC extends Spine {
    constructor(x, y, settings = {}) {
        settings.width = settings.width || 150;
        settings.height = settings.height || 200;
        settings.jsonFile = "npc.json";
        settings.atlasFile = "npc.atlas";
        settings.mixTime = 0.15;
        
        super(x, y, settings);
        
        this.body.addShape(new me.Rect(0, 0, 64, 64));
        this.body.collisionType = me.collision.types.NPC_OBJECT;
        
        this.isTalking = false;
        this.hasGreeted = false;
        
        this.setAnimation(0, "idle", null, true);
    }
    
    update(dt) {
        super.update(dt);
        this.checkPlayerProximity();
        return true;
    }
    
    checkPlayerProximity() {
        const player = me.game.world.getChildByName("player")[0];
        if (!player) return;
        
        const distance = this.pos.distance(player.pos);
        
        if (distance < 100 && !this.hasGreeted) {
            this.greet();
            this.hasGreeted = true;
        } else if (distance > 150) {
            this.hasGreeted = false;
        }
    }
    
    greet() {
        this.setAnimation(0, "wave", () => {
            this.setAnimation(0, "idle", null, true);
        }, false);
    }
    
    startDialog() {
        this.isTalking = true;
        this.setAnimation(0, "talk", null, true);
    }
    
    endDialog() {
        this.isTalking = false;
        this.setAnimation(0, "idle", null, true);
    }
    
    onCollision(response, other) {
        if (other.body.collisionType === me.collision.types.PLAYER_OBJECT) {
            if (me.input.isKeyPressed("interact") && !this.isTalking) {
                this.startDialog();
                // Show dialog UI...
            }
        }
        return false;
    }
}
```

### Enemy with AI

```javascript
import Spine from '@melonjs/spine-plugin';

export default class Enemy extends Spine {
    constructor(x, y, settings = {}) {
        settings.width = settings.width || 180;
        settings.height = settings.height || 220;
        settings.jsonFile = "enemy.json";
        settings.atlasFile = "enemy.atlas";
        
        super(x, y, settings);
        
        this.body.addShape(new me.Rect(0, 0, 64, 64));
        this.body.setMaxVelocity(2, 15);
        this.body.collisionType = me.collision.types.ENEMY_OBJECT;
        
        this.health = 100;
        this.state = "patrol";
        this.detectionRange = 200;
        this.attackRange = 50;
        this.patrolDirection = 1;
        
        this.setAnimation(0, "walk", null, true);
    }
    
    update(dt) {
        super.update(dt);
        
        const player = me.game.world.getChildByName("player")[0];
        if (player) {
            const distance = this.pos.distance(player.pos);
            
            if (distance < this.attackRange) {
                this.attack();
            } else if (distance < this.detectionRange) {
                this.chase(player);
            } else {
                this.patrol();
            }
        } else {
            this.patrol();
        }
        
        this.body.update(dt);
        return true;
    }
    
    patrol() {
        if (this.state !== "patrol") {
            this.state = "patrol";
            this.setAnimation(0, "walk", null, true);
        }
        
        this.body.force.x = this.patrolDirection * 1;
        this.flipX(this.patrolDirection < 0);
    }
    
    chase(player) {
        if (this.state !== "chase") {
            this.state = "chase";
            this.setAnimation(0, "run", null, true);
        }
        
        const direction = player.pos.x > this.pos.x ? 1 : -1;
        this.body.force.x = direction * this.body.maxVel.x;
        this.flipX(direction < 0);
    }
    
    attack() {
        if (this.state !== "attack") {
            this.state = "attack";
            this.setAnimation(0, "attack", () => {
                this.state = "idle";
                this.setAnimation(0, "idle", null, true);
            }, false);
        }
    }
    
    takeDamage(amount) {
        this.health -= amount;
        
        if (this.health <= 0) {
            this.die();
        } else {
            this.setAnimation(0, "hit", () => {
                this.setAnimation(0, "idle", null, true);
            }, false);
        }
    }
    
    die() {
        this.body.setCollisionMask(me.collision.types.NO_OBJECT);
        this.setAnimation(0, "death", () => {
            me.game.world.removeChild(this);
        }, false);
    }
    
    onCollision(response, other) {
        if (other.body.collisionType === me.collision.types.WORLD_SHAPE) {
            // Hit wall, turn around
            this.patrolDirection *= -1;
        }
        return true;
    }
}
```

### Boss with Multiple Phases

```javascript
import Spine from '@melonjs/spine-plugin';

export default class Boss extends Spine {
    constructor(x, y, settings = {}) {
        settings.width = settings.width || 400;
        settings.height = settings.height || 500;
        settings.jsonFile = "boss.json";
        settings.atlasFile = "boss.atlas";
        settings.mixTime = 0.2;
        
        super(x, y, settings);
        
        this.body.addShape(new me.Rect(0, 0, 128, 128));
        this.body.collisionType = me.collision.types.ENEMY_OBJECT;
        
        this.maxHealth = 500;
        this.health = this.maxHealth;
        this.phase = 1;
        this.state = "idle";
        this.attackTimer = 0;
        this.attackCooldown = 2000;
        
        // Start with menacing idle
        this.setAnimation(0, "idle", null, true);
    }
    
    update(dt) {
        super.update(dt);
        
        this.checkPhase();
        this.updateBehavior(dt);
        
        return true;
    }
    
    checkPhase() {
        const healthPercent = this.health / this.maxHealth;
        
        if (healthPercent <= 0.5 && this.phase === 1) {
            this.enterPhase2();
        } else if (healthPercent <= 0.25 && this.phase === 2) {
            this.enterPhase3();
        }
    }
    
    enterPhase2() {
        this.phase = 2;
        this.attackCooldown = 1500;
        this.setSkinByName("phase2");
        this.setAnimation(0, "transform", () => {
            this.setAnimation(0, "idle", null, true);
        }, false);
    }
    
    enterPhase3() {
        this.phase = 3;
        this.attackCooldown = 1000;
        this.setSkinByName("phase3");
        this.setAnimation(0, "transform", () => {
            this.setAnimation(0, "idle", null, true);
        }, false);
    }
    
    updateBehavior(dt) {
        if (this.state !== "attacking") {
            this.attackTimer += dt;
            
            if (this.attackTimer >= this.attackCooldown) {
                this.performAttack();
                this.attackTimer = 0;
            }
        }
    }
    
    performAttack() {
        this.state = "attacking";
        
        const attacks = ["slash", "smash", "roar"];
        const attack = attacks[Math.floor(Math.random() * attacks.length)];
        
        this.setAnimation(0, attack, () => {
            this.state = "idle";
            this.setAnimation(0, "idle", null, true);
        }, false);
    }
    
    takeDamage(amount) {
        this.health -= amount;
        
        if (this.health <= 0) {
            this.die();
        } else {
            // Flash/hit reaction without interrupting current animation
            // Use track 1 for overlay effects
            this.setAnimationForSkeleton(0, 1, "hit_flash", null, false);
        }
    }
    
    die() {
        this.state = "dead";
        this.body.setCollisionMask(me.collision.types.NO_OBJECT);
        
        this.setAnimation(0, "death", () => {
            // Victory sequence
            me.event.emit("boss_defeated");
            
            setTimeout(() => {
                me.game.world.removeChild(this);
            }, 2000);
        }, false);
    }
}
```

### Multi-Skeleton Customizable Character

```javascript
import Spine from '@melonjs/spine-plugin';

export default class CustomizableHero extends Spine {
    constructor(x, y, settings = {}) {
        settings.width = settings.width || 200;
        settings.height = settings.height || 300;
        
        // Load multiple skeletons for customization
        settings.skeletonConfigs = [
            { atlasFile: "body.atlas", jsonFile: "body.json", skin: "default" },
            { atlasFile: "hair.atlas", jsonFile: "hair.json", skin: "short" },
            { atlasFile: "outfit.atlas", jsonFile: "outfit.json", skin: "casual" },
            { atlasFile: "weapon.atlas", jsonFile: "weapon.json", skin: "sword" }
        ];
        
        super(x, y, settings);
        
        this.body.addShape(new me.Rect(0, 0, 64, 64));
        
        // Customization options
        this.currentBody = 0;
        this.currentHair = 1;
        this.currentOutfit = 2;
        this.currentWeapon = 3;
        
        // Set default to show all layers
        this.activeLayer = "all";
        
        // Sync animations across all skeletons
        this.syncAnimation("idle", true);
    }
    
    syncAnimation(animName, loop = true) {
        // Play same animation on all skeletons
        for (let i = 0; i < this.getSkeletonCount(); i++) {
            this.setAnimationForSkeleton(i, 0, animName, null, loop);
        }
    }
    
    changeHairStyle(styleName) {
        const hairSkeleton = this.getSkeletonAt(this.currentHair);
        if (hairSkeleton) {
            hairSkeleton.setSkinByName(styleName);
            hairSkeleton.setSlotsToSetupPose();
        }
    }
    
    changeOutfit(outfitName) {
        const outfitSkeleton = this.getSkeletonAt(this.currentOutfit);
        if (outfitSkeleton) {
            outfitSkeleton.setSkinByName(outfitName);
            outfitSkeleton.setSlotsToSetupPose();
        }
    }
    
    changeWeapon(weaponName) {
        const weaponSkeleton = this.getSkeletonAt(this.currentWeapon);
        if (weaponSkeleton) {
            weaponSkeleton.setSkinByName(weaponName);
            weaponSkeleton.setSlotsToSetupPose();
        }
    }
    
    update(dt) {
        super.update(dt);
        
        // Handle movement
        if (me.input.isKeyPressed("left")) {
            this.body.force.x = -3;
            this.flipX(true);
            this.syncAnimation("walk", true);
        } else if (me.input.isKeyPressed("right")) {
            this.body.force.x = 3;
            this.flipX(false);
            this.syncAnimation("walk", true);
        } else {
            this.syncAnimation("idle", true);
        }
        
        this.body.update(dt);
        return true;
    }
}
```

---

## 📚 API Reference Summary

### Constructor

```javascript
new Spine(x, y, settings)
```

**Key Settings:**
- `width`, `height` - Dimensions
- `atlasFile`, `jsonFile` - Spine files
- `skinName` - Default skin
- `mixTime` - Animation blend time
- `anchorPoint` - Pivot point
- `skeletonConfigs` - Array for multiple skeletons

### Essential Methods

| Method | Description |
|--------|-------------|
| `setAnimation(track, name, callback, loop)` | Play animation |
| `flipX(flip)` / `flipY(flip)` | Flip character |
| `setSkinByName(name)` | Change appearance |
| `update(dt)` | Update animation (call super) |
| `draw(renderer)` | Render (call super) |

### Physics Integration

```javascript
// Add collision body
this.body.addShape(new me.Rect(0, 0, 64, 64));

// Set physics properties
this.body.setMaxVelocity(3, 15);
this.body.setFriction(0.4, 0);

// Apply forces
this.body.force.x = velocity;
this.body.force.y = -jumpForce;

// Update physics in update()
this.body.update(dt);
```

For complete API documentation, see [Full API Reference](API.md).

---

## 🔧 Compatibility

| melonJS | @melonjs/spine-plugin | spine-runtime |
|---------|----------------------|---------------|
| v15.12.x or higher | v1.x | v4.1, v4.2-beta |

> **Note:** The current plugin is bundled with Spine runtime 4.2.x beta, which maintains backward compatibility with Spine 4.1 exports.

---

## 💡 Best Practices

### ✅ DO

```javascript
// Extend Spine directly
export default class Hero extends Spine {
    constructor(x, y, settings = {}) {
        settings.jsonFile = "hero.json";
        settings.atlasFile = "hero.atlas";
        super(x, y, settings);
    }
}

// Call super.update() and super.draw()
update(dt) {
    super.update(dt);  // ✓ Always call parent
    // Your logic here
    return true;
}

// Dispose when done
onDeactivateEvent() {
    this.dispose();
}
```

### ❌ DON'T

```javascript
// Don't extend Entity or Sprite
class Hero extends me.Entity {  // ✗ Wrong!
    constructor(x, y) {
        super(x, y);
        this.spine = new Spine(...);  // ✗ Unnecessary wrapper
    }
}

// Don't forget to call super
update(dt) {
    // Your logic
    return true;  // ✗ Missing super.update(dt)
}

// Don't forget disposal
// ✗ Memory leak if not disposed
```

---

## 🐛 Troubleshooting

### Animation not playing

```javascript
// Check available animations
console.log(this.skeleton.data.animations.map(a => a.name));

// Verify current animation
let current = this.getCurrentAnimation(0);
console.log("Playing:", current?.name);
```

### Character not visible

```javascript
// Check dimensions
console.log(this.width, this.height);

// Enable debug mode
this.debugRendering = true;

// Check z-index
console.log(this.pos.z);
```

### Physics not working

```javascript
// Make sure collision body is added
this.body.addShape(new me.Rect(0, 0, 64, 64));

// Set collision type
this.body.collisionType = me.collision.types.PLAYER_OBJECT;

// Update body in update()
this.body.update(dt);
```

---

## 📝 Complete Working Example

```javascript
import Spine from '@melonjs/spine-plugin';
import * as me from 'melonjs';

// Register plugin
me.plugin.register(SpinePlugin);

// Prepare assets
const resources = [
    { name: "hero.json", type: "spine", src: "data/hero.json" },
    { name: "hero.atlas", type: "spine", src: "data/hero.atlas" }
];

// Create player class
class Player extends Spine {
    constructor(x, y) {
        super(x, y, {
            width: 200,
            height: 300,
            jsonFile: "hero.json",
            atlasFile: "hero.atlas",
            mixTime: 0.2
        });
        
        this.body.addShape(new me.Rect(0, 0, 64, 64));
        this.body.setMaxVelocity(3, 15);
        this.setAnimation(0, "idle", null, true);
    }
    
    update(dt) {
        super.update(dt);
        
        if (me.input.isKeyPressed("left")) {
            this.body.force.x = -this.body.maxVel.x;
            this.flipX(true);
            this.setAnimation(0, "run", null, true);
        } else if (me.input.isKeyPressed("right")) {
            this.body.force.x = this.body.maxVel.x;
            this.flipX(false);
            this.setAnimation(0, "run", null, true);
        } else {
            this.setAnimation(0, "idle", null, true);
        }
        
        this.body.update(dt);
        return true;
    }
    
    draw(renderer) {
        super.draw(renderer);
    }
}

// Load and start game
me.loader.preload(resources, () => {
    const player = new Player(100, 100);
    me.game.world.addChild(player);
});
```

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