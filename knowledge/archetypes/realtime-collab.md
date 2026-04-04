# Archetype: Real-Time / Collaborative Application

## Default Stack Recommendation

| Layer | Default | Alternative | When to Switch |
|-------|---------|-------------|----------------|
| Framework | Next.js 15 (App Router) | — | Always Next.js for full-stack real-time apps |
| Language | TypeScript (strict) | — | Always |
| Styling | Tailwind CSS v4 | — | Always |
| Components | shadcn/ui | — | Always |
| Real-Time Engine | Liveblocks | PartyKit, Supabase Realtime | PartyKit for custom logic, Supabase Realtime for simple pub/sub |
| Collaboration | Yjs (via Liveblocks) | Automerge, ShareDB | Automerge for offline-first, ShareDB for OT-based |
| Database | Supabase (Postgres) | Neon | Supabase for built-in Realtime |
| ORM | Prisma | Drizzle | — |
| Auth | Clerk | NextAuth | Clerk for fast setup, orgs, avatars |
| Editor (if applicable) | Tiptap + Yjs | BlockNote, Slate | BlockNote for Notion-style, Slate for full custom |
| Hosting | Vercel + Liveblocks Cloud | Vercel + PartyKit | PartyKit if self-hosting real-time |
| Package Manager | pnpm | npm | — |

## Real-Time Technology Decision Matrix

| Solution | Best For | Latency | Complexity | Cost |
|----------|----------|---------|------------|------|
| **Liveblocks** | Collaborative editing, presence, cursors | <50ms | Low (managed) | Free 300 MAU, $99/mo for 10k |
| **PartyKit** | Custom real-time logic, game state, rooms | <50ms | Medium | Free tier, pay per compute |
| **Supabase Realtime** | Database change notifications, simple pub/sub | 100-200ms | Low | Included with Supabase |
| **Pusher / Ably** | Chat, notifications, simple events | <100ms | Low | Free tier, pay per message |
| **Socket.io** | Custom everything, self-hosted | <50ms | High (self-managed) | Free (self-hosted) |
| **Server-Sent Events** | One-way server→client, dashboards | <100ms | Low | Free |

### When to Use What

| Use Case | Recommended |
|----------|-------------|
| Collaborative document editing | **Liveblocks + Yjs** |
| Collaborative whiteboard / canvas | **Liveblocks** (storage + presence) |
| Live cursors / "who's viewing this" | **Liveblocks** (presence) |
| Chat / messaging | **Supabase Realtime** or **Pusher** |
| Live dashboard / monitoring | **Supabase Realtime** or **SSE** |
| Multiplayer game state | **PartyKit** |
| Notifications (in-app) | **Supabase Realtime** |

## Default Directory Structure

```
src/
  app/
    (app)/
      dashboard/page.tsx           # Overview of documents/rooms
      [documentId]/
        page.tsx                   # Live collaborative document/canvas
      settings/page.tsx
      layout.tsx                   # App layout with sidebar
    (marketing)/
      page.tsx                     # Landing page
      layout.tsx
    api/
      liveblocks-auth/route.ts     # Liveblocks authentication endpoint
      documents/route.ts           # CRUD for documents
      webhooks/
        liveblocks/route.ts        # Liveblocks webhook (storage events)
    layout.tsx
    globals.css
  components/
    ui/                            # shadcn/ui primitives
    collab/
      Cursors.tsx                  # Live cursor overlay
      AvatarStack.tsx              # Shows who's in the room
      PresenceIndicator.tsx        # Online/offline dot
      CollaborativeEditor.tsx      # Tiptap + Yjs editor wrapper
      SelectionHighlight.tsx       # Show what others have selected
    documents/
      DocumentCard.tsx             # Card in dashboard
      DocumentToolbar.tsx          # Toolbar (undo, redo, formatting)
    layout/
      Sidebar.tsx
      Header.tsx
  lib/
    liveblocks/
      client.ts                    # Liveblocks client config
      config.ts                    # Room config, initial storage
    db/
      schema.ts                    # Drizzle/Prisma schema
      index.ts
    utils.ts
  types/
    index.ts
  liveblocks.config.ts             # Liveblocks type definitions (Presence, Storage, etc.)
```

## Common Patterns

### Presence (Who's Here)

```typescript
// liveblocks.config.ts
type Presence = {
  cursor: { x: number; y: number } | null
  name: string
  avatar: string
  color: string        // unique color per user
  selectedId: string | null  // what element they have selected
}

// In component
const others = useOthers()       // list of other users in the room
const updateMyPresence = useUpdateMyPresence()

// Update cursor position
onPointerMove={(e) => updateMyPresence({ cursor: { x: e.clientX, y: e.clientY } })}
onPointerLeave={() => updateMyPresence({ cursor: null })}
```

### Live Cursors

```tsx
function Cursors() {
  const others = useOthers()
  return (
    <>
      {others.map(({ connectionId, presence }) => {
        if (!presence.cursor) return null
        return (
          <Cursor
            key={connectionId}
            x={presence.cursor.x}
            y={presence.cursor.y}
            name={presence.name}
            color={presence.color}
          />
        )
      })}
    </>
  )
}
```

### Collaborative Storage (CRDT via Liveblocks)

```typescript
// Define shared data structure
type Storage = {
  document: LiveObject<{
    title: string
    content: LiveList<LiveObject<Block>>
  }>
  canvas: LiveMap<string, LiveObject<Shape>>  // for whiteboard-style apps
}

// Read shared state (auto-syncs)
const title = useStorage((root) => root.document.title)

// Mutate shared state (propagates to all users)
const updateTitle = useMutation(({ storage }, newTitle: string) => {
  storage.get('document').set('title', newTitle)
}, [])
```

### Collaborative Text Editing (Tiptap + Yjs)

```typescript
import { useEditor } from '@tiptap/react'
import Collaboration from '@tiptap/extension-collaboration'
import CollaborationCursor from '@tiptap/extension-collaboration-cursor'

const editor = useEditor({
  extensions: [
    StarterKit.configure({ history: false }),  // disable local history, use Yjs
    Collaboration.configure({ document: ydoc }),
    CollaborationCursor.configure({
      provider: liveblocksProvider,
      user: { name: currentUser.name, color: currentUser.color },
    }),
  ],
})
```

### Conflict Resolution Strategy

| Approach | How | When |
|----------|-----|------|
| **CRDT (Liveblocks/Yjs)** | Automatic merge, no conflicts possible | Text editing, lists, maps |
| **Last Write Wins** | Latest timestamp wins | Simple fields (title, status) |
| **Operational Transform** | Server resolves operation order | Google Docs-style (complex, rarely DIY) |
| **Optimistic + Rollback** | Apply locally, rollback if server rejects | Form submissions, state changes |

**Recommendation:** Use Liveblocks or Yjs (CRDT-based) by default. CRDTs mathematically guarantee conflict-free merging without a central server resolving conflicts.

## Build Order

1. **Scaffolding**: Next.js, Tailwind, shadcn/ui. Install Liveblocks SDK.
2. **Auth**: Clerk setup, protected routes.
3. **Liveblocks setup**: Configure client, auth endpoint (`/api/liveblocks-auth`), type definitions.
4. **Room creation**: Create/join rooms. Store room metadata in database.
5. **Presence**: Show who's in the room (avatar stack). Implement live cursors.
6. **Collaborative storage**: Define shared data structure. Build core feature using shared state.
7. **Collaborative editor** (if applicable): Tiptap + Yjs integration, cursor colors, user names.
8. **Dashboard**: List of documents/rooms, create new, join existing.
9. **Permissions**: Room access control (owner, editor, viewer). Invite flow.
10. **Offline handling**: Detect offline, queue changes, sync on reconnect.
11. **Landing page**: Demo, features, pricing.
12. **Polish**: Connection status indicator, reconnection handling, loading states.
13. **Deploy**: Vercel, environment variables, Liveblocks production plan.

## Common Pitfalls

- **Don't build real-time from scratch with WebSockets.** Use Liveblocks, PartyKit, or at minimum Yjs. Managing WebSocket connections, reconnection, state sync, and conflict resolution is an enormous engineering effort.
- **Don't use polling for real-time features.** If users need to see changes within seconds, use WebSockets or SSE, not polling.
- **Don't forget reconnection.** Networks drop. Your app must handle disconnection gracefully — show status, buffer changes, resync on reconnect.
- **Don't send full state on every change.** Send deltas/operations only. Full state sync on initial load, incremental updates after.
- **Don't ignore user limits per room.** Free tiers have connection limits. Plan for max concurrent users per room.
- **Don't skip presence UX.** Users need to see who else is in the room and what they're doing. Without this, collaboration feels broken.

## Skills for Build Phase

| Skill | When |
|-------|------|
| `/frontend-design` | Collaborative editor UI, dashboard |
| `/shadcn-ui` | Component setup |
| `/ui-ux-pro-max` | Design system, cursor colors, presence indicators |
| `/deep-research` | CRDT vs OT comparison, Liveblocks vs alternatives |

## See Also

- `knowledge/building-blocks/frontend-patterns.md` — Supabase Realtime, state management, Client vs Server Components
- `knowledge/building-blocks/auth-patterns.md` — Room-level access control
- `knowledge/building-blocks/deployment-patterns.md` — WebSocket-compatible hosting
