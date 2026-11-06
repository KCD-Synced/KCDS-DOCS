# Carrying and attaching players

## Carrying

`carrying`, `heldBy` and `putDown()` cover carrying. `holdItem(item)` and `dropHeldItem()` are for items in the arms. Any
item the game can carry in the arms works, including the game's quest sacks, such as `rasuvUcen_sackcarryable`. When put down,
it is laid as a `GroundItem` on all screens.

## Attaching to another player

`attachTo(other, options)` keeps a body next to another player, like a rider on a horse. `detach()` ends it, and it also ends
when either player leaves. `attachedTo` reads the other player.

| Option                        | Meaning                                                                                                          |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `offset`                      | `{ x right, y forward, z up }` in metres, seen from `other`                                                      |
| `rotation`                    | degrees to turn the body; it otherwise faces the way `other` faces                                               |
| `look`                        | `'free'` (default) looks around up to 120 degrees either side; `'follow'` keeps the view on the body's direction |
| `animation`, `otherAnimation` | `{ fragment, ...playAnimation options }`, looping by default, stopping when the attachment ends                  |

`otherAnimation` is upper-body unless it says otherwise, so `other` can walk. A cuffed player keeps the cuffed pose, and
`animation` does not play on them. The attached player's game lets through only the menus, plus looking around with `'free'`.

The two keep their distance on all screens and do not collide. The attached body is carried, though, so its legs do not walk
unless you play an animation, and it passes through walls and furniture. Teleport it to a free spot if it may be inside
something when you detach it. For a prisoner walking on their own legs, use [Follow](follow.md) with a `distance`.

Reference: [Player](/docs/api/player).
