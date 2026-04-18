# Sonic 3 & Knuckles: Wario World (SHC2026) Mod Design Document

## 1. Introduction

This document outlines the design and implementation of the "Sonic 3 & Knuckles: Wario World (SHC2026)" mod for Sonic 3 A.I.R. (Angel Island Revisited) v26.03.28.0. The primary objective of this mod is to replace all instances of Dr. Eggman with Wario, including boss sprites, character sprites, and associated animations. Additionally, the mod aims to maintain compatibility with other specified mods such as Neo Boss Warnings, Pooka Party, Sonic 3 & Tenna, Pink Burner, Stone 3 A.I.R., Blue Glasses Eggman, Mecha Sonic MK.I, and Bossbar HUD.

## 2. Mod Structure

The mod adheres to the standard Sonic 3 A.I.R. modding conventions, utilizing a `mod.json` file for metadata and dependencies, and a `scripts` folder containing the core `main.lemon` script. Sprite assets are organized within a `sprites` folder.

### 2.1. `mod.json`

The `mod.json` file serves as the manifest for the mod, providing essential metadata and defining interactions with other mods. The key fields are:

*   **Metadata**: Contains general information about the mod, including its name, a unique identifier, author, description, version, and the minimum required game version.
*   **OtherMods**: Specifies dependencies and priority settings for compatibility with other mods. This ensures that the "Wario World" mod functions correctly alongside other community-made content.

### 2.2. `scripts/main.lemon`

The `main.lemon` script is the heart of the mod, responsible for intercepting and modifying game logic to implement the Wario replacement. It utilizes Lemon scripting, a language specifically designed for Sonic 3 A.I.R. modding, which allows for function overriding and custom sprite rendering.

### 2.3. `sprites/` Folder

This directory houses all the custom sprite assets for Wario. Each sprite is defined by a JSON file (e.g., `boss.json`, `character-eggman.json`) that maps sprite keys to specific regions within corresponding PNG image files (e.g., `boss.png`, `character-eggman.png`). These JSON files specify the file source, rectangular coordinates of the sprite within the image, and the sprite's center point for accurate rendering.

## 3. `main.lemon` Script Logic

The `main.lemon` script employs several key techniques to achieve the Wario replacement:

### 3.1. Global Variables

Several global variables are declared to manage Wario's state, such as `Wario.bossInit`, `Wario.laughTimer`, `Wario.lastXPos`, `Wario.golemRemaining_hits`, and `Wario.lastFrame`. These variables are crucial for controlling animations, boss behaviors, and timing of events.

### 3.2. Helper Functions

To streamline the script and improve readability, several helper functions are defined:

*   `Wario.getMetronome()`: Provides a simple metronome-like value based on the game's frame counter, often used for animating sprites that cycle through frames.
*   `Wario.getFreezableMetronome()`: A variant of `getMetronome` that pauses when the game is paused, ensuring animations don't continue during pauses.
*   `Wario.showRegular()`: Determines whether Wario should display a 
regular (non-laughing) animation based on the `laughTimer` and game over conditions.
*   `Wario.decreaseTimer()`: Decrements the `laughTimer` each frame, controlling the duration of Wario's laughing animations.

### 3.3. Main Sprite Replacement Hook (`Standalone.onWriteToSpriteTable`)

The core of the sprite replacement logic resides within the `Standalone.onWriteToSpriteTable` function. This function is a powerful hook in Sonic 3 A.I.R. that allows mods to intercept and modify sprite rendering calls before they are drawn to the screen. The script analyzes the `objA0.update_address` (which identifies the specific game object being rendered, e.g., a boss) and `objA0.animation.sprite` (the current animation frame) to determine which Wario sprite to display.

Key aspects of this function include:

*   **Boss-specific Logic**: Different `update_address` values correspond to various Dr. Eggman boss encounters (e.g., Normal Eggmobile, FBZ2 Second Boss, LRZ2 Boss, FBZ2 Mini Boss, DEZ2 Boss, LBZ2 1st Boss). For each, the script constructs a `key` string (e.g., `eggman-blue-00`, `eggman-laugh-blue-01`) that corresponds to a Wario sprite defined in the `sprites` JSON files.
*   **Animation Handling**: The script differentiates between regular animations and 
laugh animations, using `Wario.showRegular()` and `Wario.getMetronome()` to select the appropriate sprite key.
*   **Custom Sprite Drawing**: If a custom Wario sprite exists for the generated `key` (checked via `Renderer.hasCustomSprite(key)`), the `Renderer.drawCustomSprite()` function is called to render the Wario sprite instead of the original Eggman sprite. This function also handles sprite priority (`prioFlag`) and rendering flags.

### 3.4. Character Sprites (`character-eggman-blue-*`)

For instances where Dr. Eggman appears outside his Eggmobile (e.g., in cutscenes or specific level events), the script checks for corresponding `update_address` values and constructs sprite keys like `character-eggman-blue-%02x`. These keys map to Wario character sprites defined in `character-eggman.json`.

### 3.5. Ending and Savedata Screens

The mod also replaces Dr. Eggman sprites in the bad ending and Blue Sphere difficulty screens (`eggman-ending-blue-%02x`) and the savedata screen (`eggman-delete-%x%x`). The savedata screen sprite key is dynamically generated based on `objA0.animation.sprite` and a memory address, allowing for varied Wario sprites in this context.

### 3.6. Monitor Icon Replacement (`Monitor.getIconSpriteKey`)

The `Monitor.getIconSpriteKey` function is overridden to replace the Eggman monitor icon (type `0x02`) with a Wario-themed icon, specified by the `monitor_icon_eggman` key in `monitors.json`.

### 3.7. Laugh Timer Management

The script includes hooks into the `Character.BaseUpdate.Sonic`, `Character.BaseUpdate.Tails`, and `Character.BaseUpdate.Knuckles` functions to continuously decrement `Wario.laughTimer`. This ensures that Wario's laughing animations, triggered by events like getting hurt or dying, have a defined duration.

### 3.8. Boss Start Laugh Triggers

Various `address-hook` functions (e.g., `fn0691c8`, `fn0692e2`) are used to detect the start of boss encounters. When a boss fight begins, `Wario.bossInit` is set, and `Wario.BossStartLaughValue()` is called to initiate Wario's laughing animation for a specified duration.

## 4. Integration with Other Mods

The `mod.json` file includes entries under `OtherMods` to ensure compatibility and proper loading order with other mods. By setting `IsRequired` to "0" and `Priority` to "Lower" for mods like "Neo Boss Warnings", "Pooka Party", and "Bossbar HUD", this mod indicates that it can function independently but should load after these mods if they are present. This approach minimizes conflicts and allows for a harmonious modding experience.

## 5. Conclusion

The "Sonic 3 & Knuckles: Wario World (SHC2026)" mod provides a comprehensive and well-integrated replacement of Dr. Eggman with Wario in Sonic 3 A.I.R. Through careful Lemon scripting and organized sprite assets, the mod delivers a unique gameplay experience while maintaining compatibility with the broader modding ecosystem. The modular design allows for future expansion and refinement of Wario's animations and interactions within the game world.

## References

[1] Sonic 3 A.I.R. Modding Documentation. Available at: [https://raw.githubusercontent.com/Eukaryot/sonic3air/main/Oxygen/sonic3air/_master_image_template/doc/Modding.pdf](https://raw.githubusercontent.com/Eukaryot/sonic3air/main/Oxygen/sonic3air/_master_image_template/doc/Modding.pdf)
