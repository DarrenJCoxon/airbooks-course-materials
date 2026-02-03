I want to initiate the **Greenfield Full-Stack Workflow** to build a category-defining educational platform called **"AirBooks"**.

**The Vision:**
We are building a "Stateless Revision Engine" that turns static texts (like *An Inspector Calls* or the *AQA Biology Syllabus*) into interactive, AI-powered revision guides without *ever* requiring a database or user login. We call this architecture **"URL-as-Database"**.

**The Core Constraints (Non-Negotiable):**
1.  **Zero-Backend Persistence:** The entire application state (Chat History, Essay Plans, Teacher Settings) must be stored recursively in the URL hash using `lz-string` compression.
2.  **GDPR-Proof Privacy:** No student data can ever touch our persistent storage. We use a Next.js API Route strictly as a **Stateless Blind Proxy** to Scaleway (EU Inference).
3.  **Teacher "BYOK" (Bring Your Own Key):** Teachers input their API keys locally (`localStorage`) to generate "Magic Links" for their classes.
4.  **The "Engine" Pattern:** The code must be generic. It should load a "Content Pack" (JSON) so we can swap *English Lit* for *Biology* just by changing a config file.

**The "Killer Features" to Implement:**
1.  **Dictionary-Aware Compression:** To fit 3,000+ words into a URL, we must implement a custom compression engine that tokenizes domain-specific words (e.g., "Birling" -> `0x1`) before hashing.
2.  **The "Sidecar" UI:** A split-screen interface (Desktop) / Tabbed interface (Mobile) using `shadcn/ui` Resizable Panels. Left side: Source Text. Right side: AI Tutor.
3.  **Context Injection:** The proxy must verify requests and inject the invisible "Mark Scheme" from the server-side Content Pack so the AI grades correctly without student prompting.

**The Stack:**
- **Framework:** Next.js 16.x (App Router)
- **Deployment:** Vercel (Serverless Functions)
- **Styling:** Tailwind CSS + Shadcn/UI
- **Inference:** Scaleway (via OpenAI SDK)

**Please guide me through the BMad planning process in this exact order:**
1.  **Analyst**: Confirm the Project Brief with a focus on the "dictionary compression" strategy.
2.  **PM**: Generate the PRD, specifically detailing the "Recursive URL State" logic and the 4-Epic roadmap.
3.  **Architect**: Design the `ContentPack` JSON schema and the secure Proxy API contract.
4.  **UX Expert**: Define the "Sidecar" interface specs.
5.  **PO**: Validate the plan and generate the first set of User Stories.

Let's begin.
