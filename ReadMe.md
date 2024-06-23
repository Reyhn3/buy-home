# Setup

```powershell
winget install -e --id OpenJS.NodeJS
npm install npm -g
npm create svelte@latest buy-home
cd buy-home
npm install
git init && git add -A && git commit -m "chore: initial commit"
npm install svelte @sveltestrap/sveltestrap
npm install bootstrap-icons
npm run dev -- --open
```

# Pre-Render As Static

```powershell
npm install -D @sveltejs/adapter-static

# Modify svelte.config.js (see reference below)

# Add the following to src/routes/+layout.js:
export const prerender = true;

npm run build
```

## References

- https://kit.svelte.dev/docs/adapter-static
- https://kinsta.com/blog/static-sveltekit/

# Deploy

Deploy as static page to [Kinsta](https://kinsta.com/):

1. Push the code to GitLab.
2. Create an account at Kinsta.
3. **Authorize** Kinsta with your Git provider.
4. Click **Static Sites** on the left sidebar, then click **Add site**.
5. Select the repository and the branch you wish to deploy from.
6. Assign a unique name to your site.
7. Add the build settings in the following format:
   - **Build command:** `npm run build`
   - **Publish directory:** `build`
8. Finally, click **Create site**.

The code will be deployed.

## References

- https://kinsta.com/blog/static-sveltekit/

# Known Issues

- Round up is needed on boundaries
  The input controls' max value must be larger than the intended value, otherwise there's a risk that the real value cannot be set due to the step size.
  Example: `max` is set to 1025 and `step` is set to 50. The value 1025 can not be set because of the step from 1000 + 50 is greater than 1025.

# Future

- Share calculation
  Encode all parameters to a base64-string that can be shared with others.
- Set as reference
  Lock a value (maybe only the price?) and manipulate the other parameters so it is easier to compare and do "what if this changes"-scenarios.
- Explain formulas
  Show the mathematical formulas and the actual values used when explaining the fields.

# Resources

- [Svelte](https://svelte.dev/)
  Web app framework.

- [Bootstrap](https://getbootstrap.com/)
  Frontend toolkit.

  - [Bootstrap Icons](https://icons.getbootstrap.com/)

- [Sveltestrap](https://sveltestrap.js.org/?path=/docs/sveltestrap-overview--docs)

  [Source](https://github.com/sveltestrap/sveltestrap) ⤴
  Bootstrap 5 components for Svelte
