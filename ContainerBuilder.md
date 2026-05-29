# 📦 Discord.js — Components V2 Full Documentation

> A complete reference guide for building rich, modern Discord bot UIs using **Components V2** with `discord.js` v14.

---

## 📋 Table of Contents

1. [What is Components V2?](#what-is-components-v2)
2. [Requirements & Setup](#requirements--setup)
3. [Core Concept — How CV2 Works in djs](#core-concept--how-cv2-works-in-djs)
4. [Component Reference](#component-reference)
   - [ContainerBuilder](#containerbuilder)
   - [TextDisplayBuilder](#textdisplaybuilder)
   - [SeparatorBuilder](#separatorbuilder)
   - [SectionBuilder](#sectionbuilder)
   - [ThumbnailBuilder](#thumbnailbuilder)
   - [MediaGalleryBuilder](#mediagallerybuilder)
   - [ActionRowBuilder](#actionrowbuilder)
   - [ButtonBuilder](#buttonbuilder)
   - [StringSelectMenuBuilder](#stringselectmenubuilder)
5. [Sending CV2 Messages](#sending-cv2-messages)
6. [Interaction Handling](#interaction-handling)
7. [Full Examples](#full-examples)
   - [Avatar Command](#example-1-avatar-command)
   - [Help Menu with Dropdown](#example-2-help-menu-with-dropdown)
   - [User Info Card](#example-3-user-info-card)
   - [Confirmation Prompt](#example-4-confirmation-prompt)
8. [Common Patterns & Tips](#common-patterns--tips)
9. [Known Limitations](#known-limitations)

---

## What is Components V2?

**Components V2** is Discord's newer message component system that lets bots send richly structured, visually organized messages using a **layout-based approach** instead of plain embeds.

Key differences from classic embeds:

| Feature              | Classic Embeds         | Components V2             |
|----------------------|------------------------|---------------------------|
| Layout control       | Limited                | Full (containers, rows)   |
| Interactive elements | Separate from embed    | Inline inside containers  |
| Images               | image/thumbnail fields | MediaGallery component    |
| Text formatting      | Field-based            | Free-form TextDisplay     |
| Sections             | Not supported          | Section + Thumbnail       |
| Sending method       | `embeds: [...]`        | `components: [...]` + flag|

In `discord.js` v14, Components V2 uses **builders** from `discord.js` and requires a special **message flag** to enable the new rendering.

---

## Requirements & Setup

```bash
npm install discord.js
```

> ⚠️ Components V2 requires **discord.js v14.16+**. Make sure you're on the latest v14.

Your bot needs these intents at minimum:

```js
const { Client, GatewayIntentBits } = require('discord.js');

const client = new Client({
    intents: [
        GatewayIntentBits.Guilds,
        GatewayIntentBits.GuildMessages,
        GatewayIntentBits.MessageContent, // Required for prefix commands
    ]
});
```

Imports you'll use most often:

```js
const {
    ContainerBuilder,
    TextDisplayBuilder,
    SeparatorBuilder,
    SectionBuilder,
    ThumbnailBuilder,
    MediaGalleryBuilder,
    MediaGalleryItemBuilder,
    ActionRowBuilder,
    ButtonBuilder,
    StringSelectMenuBuilder,
    StringSelectMenuOptionBuilder,
    UnfurledMediaItemBuilder,
    ButtonStyle,
    SeparatorSpacingSize,
    MessageFlags,
    ComponentType,
} = require('discord.js');
```

---

## Core Concept — How CV2 Works in djs

Unlike discord.py which uses a `LayoutView` class, in discord.js you build components using **builder classes** and pass them in the `components` array of your message payload.

The critical difference: you **must** include `flags: MessageFlags.IsComponentsV2` in your send options, otherwise Discord will not render the layout correctly.

```js
const container = new ContainerBuilder()
    .addTextDisplayComponents(
        new TextDisplayBuilder().setContent("Hello, world!")
    );

await message.reply({
    components: [container],
    flags: MessageFlags.IsComponentsV2,
});
```

### The Basic Pattern

```
components: [ContainerBuilder]   ← top level, always a Container
    └── .addTextDisplayComponents()
    └── .addSeparatorComponents()
    └── .addSectionComponents()
        └── Section
            ├── text components (left)
            └── .setThumbnailAccessory() (right)
    └── .addMediaGalleryComponents()
    └── .addActionRowComponents()
        └── ActionRow
            ├── ButtonBuilder
            └── StringSelectMenuBuilder
```

---

## Component Reference

### ContainerBuilder

A **Container** is a styled box that groups all other components. It's the **only** component you add to the top-level `components` array.

```js
new ContainerBuilder()
    .setAccentColor(0x5865F2)   // Optional: hex color for left border
    .setSpoiler(false)          // Optional: hide behind spoiler blur
    .setId(1)                   // Optional: integer ID
```

**Adding children:**

```js
const container = new ContainerBuilder()
    .addTextDisplayComponents(
        new TextDisplayBuilder().setContent("Line 1"),
        new TextDisplayBuilder().setContent("Line 2"),
    )
    .addSeparatorComponents(new SeparatorBuilder())
    .addActionRowComponents(
        new ActionRowBuilder().addComponents(
            new ButtonBuilder()
                .setLabel("Click me")
                .setStyle(ButtonStyle.Primary)
                .setCustomId("btn_1")
        )
    );
```

**With accent color (colored left border):**

```js
const container = new ContainerBuilder()
    .setAccentColor(0x57F287) // Green
    .addTextDisplayComponents(
        new TextDisplayBuilder().setContent("This has a green border!")
    );
```

**Spoiler container:**

```js
const container = new ContainerBuilder()
    .setSpoiler(true)
    .addTextDisplayComponents(
        new TextDisplayBuilder().setContent("Hidden content!")
    );
```

---

### TextDisplayBuilder

Renders **markdown text** inside a layout. Supports full Discord markdown.

```js
new TextDisplayBuilder().setContent("Your text here")
```

**Markdown examples:**

```js
new TextDisplayBuilder().setContent("# Big Heading")
new TextDisplayBuilder().setContent("## Smaller Heading")
new TextDisplayBuilder().setContent("### Even Smaller")
new TextDisplayBuilder().setContent("**Bold** and *italic* and __underline__")
new TextDisplayBuilder().setContent("> This is a blockquote")
new TextDisplayBuilder().setContent("`inline code`")
new TextDisplayBuilder().setContent("-# Small subtext line")
new TextDisplayBuilder().setContent("Regular paragraph text.")
```

**Dynamic content:**

```js
new TextDisplayBuilder().setContent(`**User:** ${member.user.username}`)
new TextDisplayBuilder().setContent(`**ID:** \`${member.id}\``)
new TextDisplayBuilder().setContent(`**Joined:** <t:${Math.floor(member.joinedTimestamp / 1000)}:R>`)
```

---

### SeparatorBuilder

Adds a **horizontal divider line** between components for visual separation.

```js
new SeparatorBuilder()
    .setSpacing(SeparatorSpacingSize.Small)  // or .Large
    .setDivider(true)                        // false = invisible spacing only
```

**Usage:**

```js
const container = new ContainerBuilder()
    .addTextDisplayComponents(new TextDisplayBuilder().setContent("Section A"))
    .addSeparatorComponents(
        new SeparatorBuilder()                                      // visible, small spacing
    )
    .addTextDisplayComponents(new TextDisplayBuilder().setContent("Section B"))
    .addSeparatorComponents(
        new SeparatorBuilder().setDivider(false)                    // invisible gap only
    )
    .addTextDisplayComponents(new TextDisplayBuilder().setContent("Section C"))
    .addSeparatorComponents(
        new SeparatorBuilder().setSpacing(SeparatorSpacingSize.Large) // bigger gap
    )
    .addTextDisplayComponents(new TextDisplayBuilder().setContent("Section D"));
```

---

### SectionBuilder

A **Section** lays out text on the left and an accessory (`ThumbnailBuilder`) on the right, side-by-side.

```js
new SectionBuilder()
    .addTextDisplayComponents(
        new TextDisplayBuilder().setContent("### Welcome!"),
        new TextDisplayBuilder().setContent("Some description here.")
    )
    .setThumbnailAccessory(
        new ThumbnailBuilder()
            .setMedia(new UnfurledMediaItemBuilder().setURL("https://example.com/img.png"))
            .setDescription("Alt text")
    )
    .setId(1)  // Optional
```

**Example — Profile section:**

```js
const section = new SectionBuilder()
    .addTextDisplayComponents(
        new TextDisplayBuilder().setContent(`# ${member.displayName}`),
        new TextDisplayBuilder().setContent(`**Username:** ${member.user.username}`),
        new TextDisplayBuilder().setContent(`**ID:** \`${member.id}\``),
    )
    .setThumbnailAccessory(
        new ThumbnailBuilder()
            .setMedia(
                new UnfurledMediaItemBuilder().setURL(member.displayAvatarURL({ size: 256 }))
            )
            .setDescription(`${member.user.username}'s avatar`)
    );
```

> 💡 `SectionBuilder` is great for profile cards, help menus, and any layout where you want text next to an image.

---

### ThumbnailBuilder

A **small image** shown as an accessory inside a `SectionBuilder`.

```js
new ThumbnailBuilder()
    .setMedia(new UnfurledMediaItemBuilder().setURL("https://..."))
    .setDescription("Alt text")  // Accessibility description
    .setSpoiler(false)
```

**Using a member's avatar:**

```js
new ThumbnailBuilder()
    .setMedia(
        new UnfurledMediaItemBuilder().setURL(member.displayAvatarURL({ size: 256 }))
    )
    .setDescription(`${member.user.username}'s avatar`)
```

**Using the bot's avatar:**

```js
new ThumbnailBuilder()
    .setMedia(
        new UnfurledMediaItemBuilder().setURL(client.user.displayAvatarURL())
    )
    .setDescription("Bot icon")
```

> ⚠️ `ThumbnailBuilder` can **only** be used via `.setThumbnailAccessory()` inside a `SectionBuilder`.

---

### MediaGalleryBuilder

Displays **one or more images** in a gallery layout inside a container.

```js
new MediaGalleryBuilder()
    .addItems(
        new MediaGalleryItemBuilder()
            .setMedia(new UnfurledMediaItemBuilder().setURL("https://example.com/image.png"))
            .setDescription("Alt text")   // Optional
            .setSpoiler(false)            // Optional
    )
```

**Using an attached file:**

```js
new MediaGalleryItemBuilder()
    .setMedia(new UnfurledMediaItemBuilder().setURL("attachment://avatar.png"))
```

Then send the attachment alongside:

```js
await message.reply({
    components: [container],
    flags: MessageFlags.IsComponentsV2,
    files: [{ attachment: imageBuffer, name: "avatar.png" }],
});
```

**Multiple images:**

```js
const gallery = new MediaGalleryBuilder()
    .addItems(
        new MediaGalleryItemBuilder()
            .setMedia(new UnfurledMediaItemBuilder().setURL("https://example.com/image1.png"))
            .setDescription("Image 1"),
        new MediaGalleryItemBuilder()
            .setMedia(new UnfurledMediaItemBuilder().setURL("https://example.com/image2.png"))
            .setDescription("Image 2"),
        new MediaGalleryItemBuilder()
            .setMedia(new UnfurledMediaItemBuilder().setURL("https://example.com/image3.png"))
            .setDescription("Image 3"),
    );
```

> 💡 Up to **10 images** can be added to a single `MediaGalleryBuilder`.

---

### ActionRowBuilder

A **horizontal row** that holds interactive elements like Buttons and Select menus.

```js
new ActionRowBuilder()
    .addComponents(
        new ButtonBuilder()
            .setLabel("Yes")
            .setStyle(ButtonStyle.Success)
            .setCustomId("yes_btn"),
        new ButtonBuilder()
            .setLabel("No")
            .setStyle(ButtonStyle.Danger)
            .setCustomId("no_btn"),
    )
```

> ⚠️ An `ActionRowBuilder` can hold **up to 5 Buttons** OR **1 Select menu**, not both.

---

### ButtonBuilder

A clickable **button** inside an `ActionRowBuilder`.

```js
new ButtonBuilder()
    .setLabel("Click Me")                 // Button text
    .setStyle(ButtonStyle.Primary)        // Style (see below)
    .setCustomId("my_button")            // For interactive buttons
    .setURL("https://...")               // For link buttons only
    .setEmoji("✅")                      // Optional emoji
    .setDisabled(false)                  // Grey out button
```

**Button Styles:**

| Style                  | Appearance   | Use Case              |
|------------------------|--------------|-----------------------|
| `ButtonStyle.Primary`  | Blue         | Main action           |
| `ButtonStyle.Secondary`| Grey         | Secondary/neutral     |
| `ButtonStyle.Success`  | Green        | Confirm/positive      |
| `ButtonStyle.Danger`   | Red          | Delete/destructive    |
| `ButtonStyle.Link`     | Grey + arrow | External URL redirect |

**Interactive button (with customId):**

```js
new ButtonBuilder()
    .setLabel("Confirm")
    .setStyle(ButtonStyle.Success)
    .setCustomId("confirm_action")
```

Then listen for it:

```js
const filter = i => i.customId === "confirm_action" && i.user.id === interaction.user.id;
const collector = message.createMessageComponentCollector({ filter, time: 30_000 });

collector.on("collect", async i => {
    await i.update({ /* updated view */ });
});
```

**Link button (no collector needed):**

```js
new ButtonBuilder()
    .setLabel("Visit Website")
    .setStyle(ButtonStyle.Link)
    .setURL("https://discord.com")
```

---

### StringSelectMenuBuilder

A **dropdown menu** users can select option(s) from.

```js
new StringSelectMenuBuilder()
    .setCustomId("my_select")
    .setPlaceholder("Choose an option...")
    .setMinValues(1)
    .setMaxValues(1)
    .addOptions(
        new StringSelectMenuOptionBuilder()
            .setLabel("Option A")
            .setValue("option_a")
            .setDescription("The first option")
            .setEmoji("🔴"),
        new StringSelectMenuOptionBuilder()
            .setLabel("Option B")
            .setValue("option_b")
            .setDescription("The second option")
            .setEmoji("🟢"),
    )
```

**StringSelectMenuOptionBuilder Parameters:**

| Method            | Description                        |
|-------------------|------------------------------------|
| `.setLabel()`     | Visible text                       |
| `.setValue()`     | Internal value sent to interaction |
| `.setDescription()`| Small subtext below label         |
| `.setEmoji()`     | Emoji before label                 |
| `.setDefault()`   | Pre-selected if `true`             |

**Handling the selection:**

```js
const collector = message.createMessageComponentCollector({
    componentType: ComponentType.StringSelect,
    filter: i => i.customId === "my_select",
    time: 60_000
});

collector.on("collect", async i => {
    const selected = i.values[0]; // The chosen value
    await i.update({ /* rebuild layout with new content */ });
});
```

---

## Sending CV2 Messages

### Basic Send

```js
const container = new ContainerBuilder()
    .addTextDisplayComponents(
        new TextDisplayBuilder().setContent("Hello!")
    );

await message.reply({
    components: [container],
    flags: MessageFlags.IsComponentsV2,
});
```

### With a File Attachment

```js
const { AttachmentBuilder } = require('discord.js');

const file = new AttachmentBuilder(buffer, { name: "image.png" });

const gallery = new MediaGalleryBuilder()
    .addItems(
        new MediaGalleryItemBuilder()
            .setMedia(new UnfurledMediaItemBuilder().setURL("attachment://image.png"))
    );

const container = new ContainerBuilder()
    .addMediaGalleryComponents(gallery);

await message.reply({
    components: [container],
    flags: MessageFlags.IsComponentsV2,
    files: [file],
});
```

### Editing a CV2 Message

```js
await interaction.update({
    components: [newContainer],
    flags: MessageFlags.IsComponentsV2,
});

// Or editing a fetched message:
await targetMessage.edit({
    components: [newContainer],
    flags: MessageFlags.IsComponentsV2,
});
```

### Ephemeral CV2 Message

```js
await interaction.reply({
    components: [container],
    flags: MessageFlags.IsComponentsV2 | MessageFlags.Ephemeral,
});
```

---

## Interaction Handling

### Using a Collector (Recommended for Commands)

Collectors listen for interactions on a specific message for a set time window.

```js
const message = await channel.send({
    components: [container],
    flags: MessageFlags.IsComponentsV2,
});

// Filter: only the command author, only our button
const filter = i => i.customId === "my_btn" && i.user.id === originalUserId;

const collector = message.createMessageComponentCollector({
    filter,
    time: 60_000,  // 60 seconds
});

collector.on("collect", async interaction => {
    // Handle the interaction
    await interaction.update({
        components: [newContainer],
        flags: MessageFlags.IsComponentsV2,
    });
});

collector.on("end", async (collected, reason) => {
    if (reason === "time") {
        // Optionally disable buttons on timeout
    }
});
```

### Using client.on("interactionCreate") (For Persistent Buttons)

For buttons that should work even after a bot restart, use the global event:

```js
client.on("interactionCreate", async interaction => {
    if (!interaction.isButton()) return;

    if (interaction.customId === "confirm_action") {
        await interaction.update({ /* ... */ });
    }
});
```

### Restricting to Command Author

```js
const filter = i => {
    if (i.user.id !== originalUserId) {
        i.reply({ content: "This menu isn't for you!", flags: MessageFlags.Ephemeral });
        return false;
    }
    return true;
};
```

### Responding to Interactions

```js
// Update the message in-place (most common for CV2)
await interaction.update({
    components: [newContainer],
    flags: MessageFlags.IsComponentsV2,
});

// Send a new ephemeral reply without editing the message
await interaction.reply({
    content: "Done!",
    flags: MessageFlags.Ephemeral,
});

// Defer then follow up (for slow operations)
await interaction.deferUpdate();
// ... do async work ...
await interaction.editReply({
    components: [newContainer],
    flags: MessageFlags.IsComponentsV2,
});
```

---

## Full Examples

### Example 1: Avatar Command

Displays a user's avatar in a gallery with format download buttons.

```js
const {
    ContainerBuilder, TextDisplayBuilder, SeparatorBuilder,
    MediaGalleryBuilder, MediaGalleryItemBuilder, ActionRowBuilder,
    ButtonBuilder, ButtonStyle, MessageFlags, AttachmentBuilder,
    UnfurledMediaItemBuilder,
} = require("discord.js");
const axios = require("axios");

module.exports = {
    name: "avatar",
    aliases: ["av", "pfp"],
    async execute(message, args) {
        const member = message.mentions.members.first() || message.member;
        const avatar = member.displayAvatarURL({ size: 1024, extension: "png" });
        const avatarGif = member.displayAvatarURL({ size: 1024, extension: "gif" });
        const avatarJpg = member.displayAvatarURL({ size: 1024, extension: "jpg" });
        const avatarWebp = member.displayAvatarURL({ size: 1024, extension: "webp" });

        const isAnimated = member.user.avatar?.startsWith("a_") ?? false;
        const fetchUrl = isAnimated ? avatarGif : avatar;
        const filename = isAnimated ? "avatar.gif" : "avatar.png";

        const response = await axios.get(fetchUrl, { responseType: "arraybuffer" });
        const file = new AttachmentBuilder(Buffer.from(response.data), { name: filename });

        const gallery = new MediaGalleryBuilder()
            .addItems(
                new MediaGalleryItemBuilder()
                    .setMedia(new UnfurledMediaItemBuilder().setURL(`attachment://${filename}`))
            );

        const container = new ContainerBuilder()
            .addTextDisplayComponents(
                new TextDisplayBuilder().setContent(`# ${member.user.username}'s Avatar`)
            )
            .addSeparatorComponents(new SeparatorBuilder())
            .addTextDisplayComponents(
                new TextDisplayBuilder().setContent(`**User:** ${member.toString()}`),
                new TextDisplayBuilder().setContent(`**ID:** \`${member.id}\``),
                new TextDisplayBuilder().setContent(`**Type:** ${isAnimated ? "Animated (GIF)" : "Static"}`),
                new TextDisplayBuilder().setContent("**Size:** 1024×1024"),
            )
            .addMediaGalleryComponents(gallery)
            .addSeparatorComponents(new SeparatorBuilder())
            .addTextDisplayComponents(
                new TextDisplayBuilder().setContent("**Download formats:**")
            )
            .addActionRowComponents(
                new ActionRowBuilder().addComponents(
                    new ButtonBuilder().setLabel("PNG").setStyle(ButtonStyle.Link).setURL(avatar),
                    new ButtonBuilder().setLabel("JPG").setStyle(ButtonStyle.Link).setURL(avatarJpg),
                    new ButtonBuilder().setLabel("WEBP").setStyle(ButtonStyle.Link).setURL(avatarWebp),
                    ...(isAnimated ? [
                        new ButtonBuilder().setLabel("GIF").setStyle(ButtonStyle.Link).setURL(avatarGif)
                    ] : [])
                )
            );

        await message.reply({
            components: [container],
            flags: MessageFlags.IsComponentsV2,
            files: [file],
        });
    }
};
```

---

### Example 2: Help Menu with Dropdown

Interactive category browser using a Select menu.

```js
const {
    ContainerBuilder, TextDisplayBuilder, SeparatorBuilder,
    SectionBuilder, ThumbnailBuilder, UnfurledMediaItemBuilder,
    ActionRowBuilder, StringSelectMenuBuilder, StringSelectMenuOptionBuilder,
    MessageFlags, ComponentType,
} = require("discord.js");

const COMMANDS = {
    "Moderation": ["`ban`", "`kick`", "`mute`", "`warn`", "`purge`"],
    "Utility":    ["`avatar`", "`userinfo`", "`serverinfo`", "`ping`"],
    "Fun":        ["`8ball`", "`meme`", "`joke`", "`rps`"],
    "Economy":    ["`balance`", "`daily`", "`work`", "`shop`"],
};

function buildHomeContainer(client) {
    const select = new StringSelectMenuBuilder()
        .setCustomId("help_select")
        .setPlaceholder("Select a category...")
        .addOptions(
            Object.entries(COMMANDS).map(([cat, cmds]) =>
                new StringSelectMenuOptionBuilder()
                    .setLabel(cat)
                    .setValue(cat)
                    .setDescription(`${cmds.length} commands`)
            )
        );

    const section = new SectionBuilder()
        .addTextDisplayComponents(
            new TextDisplayBuilder().setContent("### 🤖 Bot Help"),
            new TextDisplayBuilder().setContent("Use the dropdown below to browse commands."),
        )
        .setThumbnailAccessory(
            new ThumbnailBuilder()
                .setMedia(new UnfurledMediaItemBuilder().setURL(client.user.displayAvatarURL()))
                .setDescription("Bot avatar")
        );

    return new ContainerBuilder()
        .addSectionComponents(section)
        .addSeparatorComponents(new SeparatorBuilder())
        .addTextDisplayComponents(
            new TextDisplayBuilder().setContent(`-# ${Object.keys(COMMANDS).length} categories available`)
        )
        .addActionRowComponents(
            new ActionRowBuilder().addComponents(select)
        );
}

function buildCategoryContainer(client, category) {
    const cmds = COMMANDS[category];

    const select = new StringSelectMenuBuilder()
        .setCustomId("help_select")
        .setPlaceholder("Select a category...")
        .addOptions(
            Object.entries(COMMANDS).map(([cat, c]) =>
                new StringSelectMenuOptionBuilder()
                    .setLabel(cat)
                    .setValue(cat)
                    .setDescription(`${c.length} commands`)
                    .setDefault(cat === category)
            )
        );

    const section = new SectionBuilder()
        .addTextDisplayComponents(
            new TextDisplayBuilder().setContent(`### 📂 ${category}`),
            new TextDisplayBuilder().setContent(cmds.join("  ")),
        )
        .setThumbnailAccessory(
            new ThumbnailBuilder()
                .setMedia(new UnfurledMediaItemBuilder().setURL(client.user.displayAvatarURL()))
                .setDescription(`${category} icon`)
        );

    return new ContainerBuilder()
        .addSectionComponents(section)
        .addSeparatorComponents(new SeparatorBuilder())
        .addTextDisplayComponents(
            new TextDisplayBuilder().setContent("-# Use the dropdown to switch categories.")
        )
        .addActionRowComponents(
            new ActionRowBuilder().addComponents(select)
        );
}

module.exports = {
    name: "help",
    async execute(message, args, client) {
        const container = buildHomeContainer(client);

        const reply = await message.reply({
            components: [container],
            flags: MessageFlags.IsComponentsV2,
        });

        const collector = reply.createMessageComponentCollector({
            componentType: ComponentType.StringSelect,
            filter: i => {
                if (i.user.id !== message.author.id) {
                    i.reply({ content: "Run `help` yourself to use this menu.", flags: MessageFlags.Ephemeral });
                    return false;
                }
                return true;
            },
            time: 120_000,
        });

        collector.on("collect", async i => {
            const category = i.values[0];
            const newContainer = buildCategoryContainer(client, category);

            await i.update({
                components: [newContainer],
                flags: MessageFlags.IsComponentsV2,
            });
        });
    }
};
```

---

### Example 3: User Info Card

A clean profile card using Section + Thumbnail.

```js
const {
    ContainerBuilder, TextDisplayBuilder, SeparatorBuilder,
    SectionBuilder, ThumbnailBuilder, UnfurledMediaItemBuilder,
    MessageFlags,
} = require("discord.js");

module.exports = {
    name: "userinfo",
    aliases: ["ui", "whois"],
    async execute(message, args) {
        const member = message.mentions.members.first() || message.member;
        const user = member.user;

        const joined  = `<t:${Math.floor(member.joinedTimestamp / 1000)}:R>`;
        const created = `<t:${Math.floor(user.createdTimestamp / 1000)}:R>`;
        const roles   = member.roles.cache.filter(r => r.name !== "@everyone");
        const topRole = roles.sort((a, b) => b.position - a.position).first();
        const roleList = roles.size > 0
            ? [...roles.values()].slice(0, 5).map(r => r.toString()).join(", ")
              + (roles.size > 5 ? " ..." : "")
            : "None";

        const accentColor = member.displayColor || 0x5865F2;

        const section = new SectionBuilder()
            .addTextDisplayComponents(
                new TextDisplayBuilder().setContent(`# ${member.displayName}`),
                new TextDisplayBuilder().setContent(`**Username:** ${user.username}`),
                new TextDisplayBuilder().setContent(`**ID:** \`${member.id}\``),
                new TextDisplayBuilder().setContent(`**Bot:** ${user.bot ? "Yes" : "No"}`),
            )
            .setThumbnailAccessory(
                new ThumbnailBuilder()
                    .setMedia(
                        new UnfurledMediaItemBuilder().setURL(
                            member.displayAvatarURL({ size: 256 })
                        )
                    )
                    .setDescription(`${user.username}'s avatar`)
            );

        const container = new ContainerBuilder()
            .setAccentColor(accentColor)
            .addSectionComponents(section)
            .addSeparatorComponents(new SeparatorBuilder())
            .addTextDisplayComponents(
                new TextDisplayBuilder().setContent(`**Joined Server:** ${joined}`),
                new TextDisplayBuilder().setContent(`**Account Created:** ${created}`),
                new TextDisplayBuilder().setContent(`**Top Role:** ${topRole ? topRole.toString() : "None"}`),
                new TextDisplayBuilder().setContent(`**Roles (${roles.size}):** ${roleList}`),
            );

        await message.reply({
            components: [container],
            flags: MessageFlags.IsComponentsV2,
        });
    }
};
```

---

### Example 4: Confirmation Prompt

A yes/no prompt with dynamic layout updates on button click.

```js
const {
    ContainerBuilder, TextDisplayBuilder, SeparatorBuilder,
    ActionRowBuilder, ButtonBuilder, ButtonStyle,
    MessageFlags, ComponentType,
} = require("discord.js");

function buildPrompt(action) {
    return new ContainerBuilder()
        .setAccentColor(0xFEE75C) // Yellow
        .addTextDisplayComponents(
            new TextDisplayBuilder().setContent("## ⚠️ Are you sure?")
        )
        .addSeparatorComponents(new SeparatorBuilder())
        .addTextDisplayComponents(
            new TextDisplayBuilder().setContent(`You are about to: **${action}**`),
            new TextDisplayBuilder().setContent("-# This action cannot be undone."),
        )
        .addSeparatorComponents(new SeparatorBuilder())
        .addActionRowComponents(
            new ActionRowBuilder().addComponents(
                new ButtonBuilder()
                    .setLabel("✅ Confirm")
                    .setStyle(ButtonStyle.Success)
                    .setCustomId("confirm"),
                new ButtonBuilder()
                    .setLabel("❌ Cancel")
                    .setStyle(ButtonStyle.Danger)
                    .setCustomId("cancel"),
            )
        );
}

function buildResult(confirmed, action) {
    return new ContainerBuilder()
        .setAccentColor(confirmed ? 0x57F287 : 0xED4245) // Green or Red
        .addTextDisplayComponents(
            new TextDisplayBuilder().setContent(
                confirmed ? "## ✅ Confirmed!" : "## ❌ Cancelled"
            ),
            new TextDisplayBuilder().setContent(
                confirmed
                    ? `Action executed: **${action}**`
                    : "No changes were made."
            ),
        );
}

module.exports = {
    name: "nuke",
    async execute(message, args) {
        if (!message.member.permissions.has("Administrator")) {
            return message.reply("You need Administrator permission.");
        }

        const action = `Nuke #${message.channel.name}`;

        const reply = await message.reply({
            components: [buildPrompt(action)],
            flags: MessageFlags.IsComponentsV2,
        });

        const filter = i => {
            if (i.user.id !== message.author.id) {
                i.reply({ content: "Not your prompt!", flags: MessageFlags.Ephemeral });
                return false;
            }
            return ["confirm", "cancel"].includes(i.customId);
        };

        const collector = reply.createMessageComponentCollector({
            componentType: ComponentType.Button,
            filter,
            time: 30_000,
            max: 1,
        });

        collector.on("collect", async i => {
            const confirmed = i.customId === "confirm";
            await i.update({
                components: [buildResult(confirmed, action)],
                flags: MessageFlags.IsComponentsV2,
            });

            if (confirmed) {
                // Do the actual action here
                console.log("Nuke confirmed!");
            }
        });

        collector.on("end", async (collected, reason) => {
            if (reason === "time" && collected.size === 0) {
                await reply.edit({
                    components: [buildResult(false, action)],
                    flags: MessageFlags.IsComponentsV2,
                });
            }
        });
    }
};
```

---

## Common Patterns & Tips

### ✅ Always pass `flags: MessageFlags.IsComponentsV2`

This is the most common mistake. Without this flag, your layout won't render as CV2.

```js
// ✅ Correct
await message.reply({
    components: [container],
    flags: MessageFlags.IsComponentsV2,
});

// ❌ Wrong — will not render properly
await message.reply({
    components: [container],
});
```

### ✅ Combining IsComponentsV2 with Ephemeral

Use the bitwise OR `|` operator to combine flags:

```js
await interaction.reply({
    components: [container],
    flags: MessageFlags.IsComponentsV2 | MessageFlags.Ephemeral,
});
```

### ✅ Using attachment:// for local images

```js
const file = new AttachmentBuilder(buffer, { name: "image.png" });

const gallery = new MediaGalleryBuilder()
    .addItems(
        new MediaGalleryItemBuilder()
            .setMedia(new UnfurledMediaItemBuilder().setURL("attachment://image.png"))
    );

await message.reply({
    components: [container],
    flags: MessageFlags.IsComponentsV2,
    files: [file],
});
```

### ✅ Accent colors on containers

```js
// Discord named color
new ContainerBuilder().setAccentColor(0x5865F2)  // Blurple

// Member's role color
new ContainerBuilder().setAccentColor(member.displayColor || 0x5865F2)

// Custom RGB (convert to hex int)
new ContainerBuilder().setAccentColor(0xFF6400)  // Orange
```

### ✅ Discord timestamps in TextDisplay

```js
const ts = Math.floor(Date.now() / 1000);

new TextDisplayBuilder().setContent(`Full date: <t:${ts}:F>`)
new TextDisplayBuilder().setContent(`Relative: <t:${ts}:R>`)   // "3 minutes ago"
new TextDisplayBuilder().setContent(`Date only: <t:${ts}:D>`)
new TextDisplayBuilder().setContent(`Time only: <t:${ts}:t>`)
```

### ✅ Disable buttons after collector ends

```js
collector.on("end", async (collected, reason) => {
    if (reason === "time") {
        await reply.edit({
            components: [
                new ContainerBuilder()
                    .addTextDisplayComponents(
                        new TextDisplayBuilder().setContent("This menu has expired.")
                    )
                    .addActionRowComponents(
                        new ActionRowBuilder().addComponents(
                            new ButtonBuilder()
                                .setLabel("Expired")
                                .setStyle(ButtonStyle.Secondary)
                                .setCustomId("expired")
                                .setDisabled(true)
                        )
                    )
            ],
            flags: MessageFlags.IsComponentsV2,
        });
    }
});
```

### ✅ Builder functions keep code clean

Instead of building containers inline, break them into functions:

```js
function buildProfileContainer(member) {
    return new ContainerBuilder()
        .addTextDisplayComponents(
            new TextDisplayBuilder().setContent(`# ${member.displayName}`)
        );
}

function buildErrorContainer(message) {
    return new ContainerBuilder()
        .setAccentColor(0xED4245)
        .addTextDisplayComponents(
            new TextDisplayBuilder().setContent(`❌ ${message}`)
        );
}

// Usage
await channel.send({
    components: [buildProfileContainer(member)],
    flags: MessageFlags.IsComponentsV2,
});
```

---

## Known Limitations

| Limitation | Details |
|------------|---------|
| No nested Containers | `ContainerBuilder` cannot be placed inside another `ContainerBuilder` |
| Thumbnail is Section-only | `ThumbnailBuilder` can only be used via `.setThumbnailAccessory()` in `SectionBuilder` |
| ActionRow limits | Max 5 Buttons OR 1 Select per ActionRow |
| MediaGallery limit | Up to 10 images per gallery |
| Flag is mandatory | Must pass `flags: MessageFlags.IsComponentsV2` or layout won't render |
| No mixing with embeds | Cannot use `embeds: [...]` and CV2 components in the same message |
| Select in ActionRow | A Select must be the **only** item in its ActionRow |
| Top-level only Container | Only `ContainerBuilder` goes in the top-level `components` array |

---

> 📝 **Tip:** Components V2 messages cannot be mixed with classic embeds. Once you go CV2 on a message, skip the `embeds` field entirely.

---

*Made with ❤️ by NaAz (Zerohost356) for discord.js bot developers*
