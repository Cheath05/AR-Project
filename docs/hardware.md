# Hardware, for Stage 13

Nothing here needs buying until August 2027. Prices are from searches on 9 November 2026, and sources disagreed on several, so re-check before buying.

## The honest picture

No single device is both true spatial AR and comfortable all day at a student budget. There are three families, and they are good at different things:

| Family | What it does well | What it does badly |
|---|---|---|
| **Passthrough headsets** (Quest) | Real spatial AR: hand tracking, room understanding, anchors, big field of view. By far the most tutorials | Bulky; around 2 hours of battery; not something you wear to class |
| **Display glasses** (Xreal, Viture, RayNeo) | A big floating screen in light glasses | Little or no understanding of the room; limited developer access |
| **HUD / AI glasses** (Brilliant Labs, Mentra, Even) | All-day wearable; open SDKs; good for notifications and an assistant | Tiny or no display; no spatial tracking |

## Devices

| Device | ~Price | Family | Developer access | Notes |
|---|---|---|---|---|
| **Meta Quest 3S** | $350 | Headset | Excellent: Unity, Unreal, WebXR | Same passthrough and SDK as Quest 3; no depth sensor. Best value |
| **Meta Quest 3** | $599 | Headset | Excellent | Adds a depth sensor, sharper lenses, wider view. Best learning headset |
| **Xreal Aura** | $1,099 intro, $1,279 list | Android XR glasses | Android XR SDK, Unity | First Android XR glasses, shipping mid-November 2026. Closest to all-day spatial AR, but new and expensive |
| Xreal 1S | $449 | Display glasses | Xreal SDK (Unity) | 3DoF head tracking built in |
| Xreal One | ~$499 | Display glasses | Xreal SDK (Unity) | 6DoF needs the separate Eye camera add-on |
| Brilliant Labs Halo | $399 | HUD glasses | Open source, Python/Lua | Good target for the daily HUD and assistant |
| Brilliant Labs Frame | $349 | HUD glasses | Open source, Python/Lua | Older; less active than Halo |
| Mentra Live | $349–449 | Camera glasses | Open source, TypeScript | No display; camera and audio assistant |
| RayNeo Air 4 Pro | $299 | Display glasses | Minimal | A screen, not a dev platform |
| Viture Pro 2 | $299 | Display glasses | Minimal | A screen, not a dev platform |
| Even Realities G1 | $499–600 | HUD glasses | No official SDK | Not recommended for development |

## Recommendation

Decide after Stage 11.

- **Staying under $500:** Quest 3S. Everything in stages 10–12 ports to it, and hand input is where the idea board starts to feel physical.
- **Stretching:** Quest 3, for the depth sensor (better occlusion and room meshing, the Stage 6 concepts).
- **Xreal Aura:** wait. By August 2027 the price, reviews and Android XR tooling will be much clearer.
- **For all-day wear later:** a HUD pair such as Halo as a second device for the assistant, once it works on the phone.

## Working with a Quest from your machines

- **Windows PC:** Quest Link lets you press Play in Unity and see the scene in the headset immediately. The fastest way to iterate.
- **Mac:** Link is not supported. You build to the headset each time, or test in the Meta XR Simulator (Apple Silicon only).

## What you already have

An iPhone running ARKit covers stages 0–12. If yours is a Pro model with LiDAR, Stage 6 (meshing and occlusion) will work much better; without LiDAR, the brief has a fallback.

## Developer docs

- Meta Quest: https://developers.meta.com/horizon/documentation/unity/unity-project-setup
- Android XR: https://developer.android.com/develop/xr
- Xreal: https://docs.xreal.com
- Brilliant Labs: https://docs.brilliant.xyz
- Mentra: https://docs.mentra.glass

## Price sources

- https://vr.org/quest-3-vs-quest-3s
- https://vrarwiki.com/wiki/Meta_Quest_3S
- https://roadtovr.com/google-ar-glasses-xreal-aura-release-date-price/
- https://oxivue.com/guides/open-source-smart-glasses
