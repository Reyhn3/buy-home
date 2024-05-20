```powershell
winget install -e --id OpenJS.NodeJS
npm install npm -g
npm create svelte@latest buy-home
cd buy-home
npm install
git init && git add -A && git commit -m "chore: initial commit"
npm run dev -- --open
# npm install bootstrap@latest
npm install svelte @sveltestrap/sveltestrap
npm install bootstrap-icons
npm run build
```

# Pre-Render As Static

```powershell
npm install -D @sveltejs/adapter-static

# Modify svelte.config.js

npm run build
```

## References

- https://kit.svelte.dev/docs/adapter-static
- https://kinsta.com/blog/static-sveltekit/

# Resources

- [Svelte](https://svelte.dev/)
  Web app framework.

- [Bootstrap](https://getbootstrap.com/)
  Frontend toolkit.

  - [Bootstrap Icons](https://icons.getbootstrap.com/)

- [Sveltestrap](https://sveltestrap.js.org/?path=/docs/sveltestrap-overview--docs)

  [Source](https://github.com/sveltestrap/sveltestrap) ⤴
  Bootstrap 5 components for Svelte
