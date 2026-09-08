# YOD Web Platform

Core web storefront and interactive 3D apparel visualization engine for the **YOD** unisex streetwear brand.

---

## 🛠 Tech Stack

- **Frontend:** Next.js (App Router), TypeScript, Tailwind CSS
- **3D Visualization:** Three.js / React Three Fiber (`@react-three/fiber`), `@react-three/drei`
- **State Management:** Zustand
- **Backend / API:** Node.js, Next.js Route Handlers
- **Database & Storage:** PostgreSQL (Prisma ORM), Cloud Object Storage (CDN-backed GLB assets)
- **Tooling & CI/CD:** ESLint, Prettier, GitHub Actions, Docker

---

## 📁 Repository Structure

```text
├── apps/
│   └── web/
│       ├── public/
│       │   ├── models/            # Draco-compressed .glb garment assets
│       │   └── textures/          # Normal/roughness PBR maps
│       ├── src/
│       │   ├── app/               # Next.js App Router (pages & API routes)
│       │   ├── components/
│       │   │   ├── 3d/            # Canvas, CanvasStage, Mesh viewers, Stitch shaders
│       │   │   ├── cart/          # Cart drawer, bag slide-overs
│       │   │   ├── common/        # Buttons, modals, form controls
│       │   │   └── product/       # PDP layout, size selector, specs drawer
│       │   ├── hooks/             # Custom React hooks (use3DViewer, useCart)
│       │   ├── lib/               # Prisma client, API utilities, Three.js loaders
│       │   ├── store/             # Zustand stores (viewerState, cartState)
│       │   └── types/             # TypeScript schema definitions
├── .env.example
├── docker-compose.yml
├── package.json
└── README.md
