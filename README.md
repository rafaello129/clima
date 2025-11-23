# Weather App - App Clima

A modern, responsive weather application built with React and TypeScript that displays current weather conditions, forecasts, and air quality information for multiple cities in Mexico.

## 🌐 Live Demo

The app is deployed on GitHub Pages: [https://rafaello129.github.io/clima](https://rafaello129.github.io/clima)

## 📖 Documentation

**For Backend Development Team**: See [ESTRUCTURA_PROYECTO.md](./ESTRUCTURA_PROYECTO.md) for comprehensive documentation of the frontend architecture (in Spanish). This document includes:
- Complete project structure
- API integration details
- TypeScript interfaces and data types
- Data flow diagrams
- Suggested backend API endpoints
- Authentication flow recommendations

## 🚀 Quick Start

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

## Available Scripts

In the project directory, you can run:

### `npm start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in the browser.

The page will reload if you make edits.\
You will also see any lint errors in the console.

### `npm test`

Launches the test runner in the interactive watch mode.\
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `npm run build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

### `npm run deploy`

Deploys the app to GitHub Pages.\
This script builds the app and pushes it to the `gh-pages` branch.

**Note:** The app is also automatically deployed via GitHub Actions whenever changes are pushed to the `main` branch.

### `npm run eject`

**Note: this is a one-way operation. Once you `eject`, you can’t go back!**

If you aren’t satisfied with the build tool and configuration choices, you can `eject` at any time. This command will remove the single build dependency from your project.

Instead, it will copy all the configuration files and the transitive dependencies (webpack, Babel, ESLint, etc) right into your project so you have full control over them. All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them. At this point you’re on your own.

You don’t have to ever use `eject`. The curated feature set is suitable for small and middle deployments, and you shouldn’t feel obligated to use this feature. However we understand that this tool wouldn’t be useful if you couldn’t customize it when you are ready for it.

## Learn More

You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).

To learn React, check out the [React documentation](https://reactjs.org/).

## 🚀 Deployment

This project is configured for automatic deployment to GitHub Pages:

1. **Automatic Deployment**: Pushing to the `main` branch triggers a GitHub Actions workflow that builds and deploys the app automatically.

2. **Manual Deployment**: You can also deploy manually using:
   ```bash
   npm run deploy
   ```

### First-time Setup for GitHub Pages

To enable GitHub Pages for this repository:

1. Go to repository Settings → Pages
2. Under "Build and deployment":
   - Source: Select "GitHub Actions"
3. The app will be available at: `https://rafaello129.github.io/clima`

### Configuration

The app is configured with:
- **Homepage**: `https://rafaello129.github.io/clima` (in package.json)
- **Basename**: `/clima` (in BrowserRouter)
- **GitHub Actions**: Workflow in `.github/workflows/deploy.yml`

