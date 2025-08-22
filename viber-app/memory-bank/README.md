# 📝 Task Manager App

A fast, accessible, single-user task manager built with Next.js, TypeScript, and Material UI.  
All data lives in your browser — no backend, no sign-in, just productivity!

---

## 🚀 Quickstart

1. **Clone the repo:**  
   `git clone <your-repo-url> && cd <repo-folder>`

2. **Install dependencies:**  
   `pnpm install`

3. **Start the app:**  
   `pnpm dev`

4. **Open in browser:**  
   [http://localhost:3000](http://localhost:3000)

---

## 🛠️ Tech Stack

- Next.js (App Router)
- TypeScript (strict)
- Material UI (MUI)
- LocalStorage (v1)
- Vercel (deployment)

---

## ✨ Features

- ✅ Create, edit, delete, and complete tasks
- 🗂️ Task details: title, description, due date, priority
- 🌗 Light/dark mode (system + manual toggle, persists)
- 📱 Responsive UI
- 🧠 Data and theme persist across reloads
- ♿️ Accessibility & keyboard navigation
- 🧹 Linting, formatting, basic tests

---

## 📦 Scripts

- `pnpm dev` — Start development server
- `pnpm build` — Build for production
- `pnpm lint` — Run ESLint
- `pnpm typecheck` — TypeScript strict check
- `pnpm format` — Format code with Prettier
- `pnpm test` — Run unit/component tests

---

## 🗂️ Project Structure

```
src/
  app/
  components/
  context/
  hooks/
  lib/
  styles/
```

---

## 🧑‍💻 Architecture

- **State:** React Context + Reducer
- **Persistence:** LocalStorage (v1), optional IndexedDB (v2)
- **Theming:** MUI ThemeProvider, system preference detection
- **Accessibility:** ARIA, keyboard, color contrast

---

## 📋 Roadmap

- [x] MVP: CRUD, theming, persistence, a11y
- [ ] Sorting & filtering
- [ ] Import/export tasks
- [ ] PWA support
- [ ] Custom categories/tags

---

## 📝 ADRs & Decisions

- No backend/auth for v1
- Strict TypeScript everywhere
- Minimal dependencies for fast load
- Accessibility is non-negotiable

---

## 🧑‍🏫 Onboarding

1. Clone, install, and run — see Quickstart above.
2. Review [`task_manager_app_extremely_granular_step_by_step_implementation_plan.md`](./memory-bank/task_manager_app_extremely_granular_step_by_step_implementation_plan.md) for step-by-step guidance.
3. All major logic is in `src/context`, `src/lib`, and `src/components`.

---

## 📄 License

MIT

---

## 🤝 Contributing

PRs welcome! Please follow code style and add tests for new features.

---

## 🏁 Deploy

- Vercel: Connect repo, auto-deploy on push to main.

---

## 💡 Limitations

- Single user only
- No cloud sync or multi-device support (yet)
- Data stored locally in browser

---

## 📬 Contact

Questions or feedback? Open an issue or reach out via GitHub!
