# Decisions

User answers to clarifying questions, recorded at submit time.
- [Data Layer] Should these screens be static/hardcoded presentation mockups (sample data baked into HTML), or wired to a real Zoho Creator backend with live forms, reports, and workflows? -> Funtional app with modern and polished UI hosted in Creator. (custom answer)
- [React → HTML] The BRD specifies 'React with inline pixel-value styles' — since we're building in Zoho Creator (not a React app), should I interpret this as: use raw HTML pages with inline pixel styles throughout (no Tailwind, no ZCS component library), matching the design intent? -> I want a polished, modern UI. The end result should be funtionally working app. Optimize for that. (custom answer)
- [Scope] Which modules should I build in this session? -> All three - Modules for the App "Zylker Budget Approval" (custom answer)
- [Approvals UX] For Module 2 (Approvals), the detail panel shows one selected request at a time. How should switching between queue rows and detail work? -> Click row → show detail below/beside
