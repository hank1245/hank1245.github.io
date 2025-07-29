---
layout: post
title: Pre-commit Validation with Husky and lint-staged
subtitle: Automating Code Quality in JavaScript Projects
tags: [JavaScript, TypeScript, DevTools, Git]
comments: true
author: Hank Kim
---

# Pre-commit Validation with Husky and lint-staged

I've been working with React Native and TypeScript for a while now, and one thing that constantly frustrated me was seeing inconsistent code styles creeping into our repository. Even though we had ESLint rules set up, team members (myself included) would sometimes forget to run linting before committing changes. The result? A messy git history filled with "fix linting errors" commits.

After dealing with this pain point for months, I decided to implement an automated solution using Husky and lint-staged. The setup transformed our development workflow, and I want to share exactly how I did it and the lessons I learned along the way.

## Why Pre-commit Validation?

Pre-commit validation ensures that code adheres to your project's linting rules before it enters the repository. Instead of relying on developers to remember to run linting manually, we can automate this process to:

- Prevent commits that violate ESLint rules
- Automatically format code using Prettier
- Maintain consistent code quality across the team
- Catch issues early in the development process

## The Tools: Husky and lint-staged

When I first heard about Git hooks, I was intimidated by the setup process. Traditional Git hooks require creating shell scripts in the `.git/hooks` directory, which felt clunky and wasn't version-controlled with the project.

**Husky** changed everything for me. It's a tool that makes Git hooks incredibly easy to set up and manage. Instead of dealing with raw shell scripts, Husky lets you define hooks in your `package.json` or in simple files that get committed with your project. When you run `git commit`, Husky automatically triggers the scripts you've defined.

**lint-staged** was the missing piece of the puzzle. Initially, I tried running ESLint on the entire codebase during pre-commit, but it was painfully slow. lint-staged solved this by only running linters on files that are actually staged for commit. This makes the process lightning-fast and keeps you focused on the changes you're actually making.

The combination of these two tools creates a seamless experience where code quality checks happen automatically, but only when and where they're needed.

## Setup Process

### 1. Installation

The installation process is straightforward, but I learned a few things along the way. Here's what you need:

```bash
npm install husky lint-staged prettier
npm init @eslint/config
```

**Pro tip from my experience**: When running `npm init @eslint/config`, make sure to select "TypeScript" if you're using it, and choose "React" for the framework. The setup wizard will automatically configure the right plugins and rules, saving you from manually figuring out the configuration later.

### 2. Configure lint-staged

This is where I spent the most time tweaking settings. Add the following configuration to your `package.json`:

```json
{
  "lint-staged": {
    "./**/*.{js,ts,jsx,tsx,html,css}": "prettier --write",
    "./**/*.{js,ts,jsx,tsx}": "eslint"
  }
}
```

Let me explain what each part does based on my testing:

- **Prettier with `--write`**: This automatically formats your code and saves the changes. I initially forgot the `--write` flag and couldn't figure out why my code wasn't being formatted!
- **File patterns**: The `/**/*.{...}` pattern ensures we catch files in any subdirectory. I learned this the hard way when my components in nested folders weren't being linted.
- **Separate ESLint rule**: I run ESLint only on JS/TS files because running it on CSS files would cause errors.

**Important lesson**: The order matters! Prettier runs first to format the code, then ESLint runs to check for issues. If you reverse this order, ESLint might complain about formatting that Prettier would have fixed.

### 3. Setting up Husky

Here's where it gets interesting. Initialize Husky and create a pre-commit hook:

```bash
npx husky-init && npm install
npx husky add .husky/pre-commit "npm test"
```

The `husky-init` command creates a `.husky` directory and sets up the basic structure. By default, it creates a pre-commit hook that runs `npm test`, but we want to run our linting instead.

Edit the `.husky/pre-commit` file to look like this:

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx lint-staged
```

**Debugging tip**: When I first set this up, I had issues with the hook not running. Make sure the `.husky/pre-commit` file is executable. On macOS/Linux, you can check this with `ls -la .husky/pre-commit`. If it doesn't have execute permissions, run `chmod +x .husky/pre-commit`.

## How It Works in Practice

Let me walk you through what actually happens when you try to commit code. I'll use a real example from when I was working on a React Native component:

1. **I stage my changes**: `git add src/components/UserProfile.tsx`
2. **I attempt to commit**: `git commit -m "add user profile component"`
3. **Husky intercepts**: The pre-commit hook triggers automatically
4. **lint-staged takes over**: It only looks at `UserProfile.tsx` (the staged file)
5. **Prettier runs first**: Formats the code and saves changes
6. **ESLint runs second**: Checks for linting issues
7. **Two possible outcomes**:
   - ✅ **Success**: If everything passes, the commit goes through
   - ❌ **Failure**: If ESLint finds errors, the commit is blocked with detailed error messages

The beautiful thing is that if Prettier makes formatting changes, those changes are automatically included in your commit. No need to manually stage them again!

## Common Issues I Encountered (And How I Fixed Them)

### The Dreaded React Version Warning

This was the first issue that hit me. ESLint kept complaining: "Warning: React version not specified in eslint-plugin-react settings." 

The fix is simple - add this to your `.eslintrc.json`:

```json
{
  "settings": {
    "react": {
      "version": "detect"
    }
  },
  "rules": {
    "react/react-in-jsx-scope": ["off"],
    "react/jsx-props-no-spreading": ["off"],
    "react/jsx-uses-react": ["off"]
  }
}
```

### Complete ESLint Configuration Example

Here's a complete ESLint configuration for a TypeScript React project:

```json
{
  "env": {
    "browser": true,
    "es2021": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  "overrides": [],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "plugins": ["react", "@typescript-eslint"],
  "rules": {
    "react/react-in-jsx-scope": ["off"],
    "react/jsx-props-no-spreading": ["off"],
    "react/jsx-uses-react": ["off"]
  },
  "settings": {
    "react": {
      "version": "detect"
    }
  }
}
```

## TypeScript Configuration

For TypeScript projects, ensure your `tsconfig.json` extends the proper base configuration:

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true
  }
}
```

## Benefits and Best Practices

### Benefits
- **Consistency**: All committed code follows the same formatting and linting rules
- **Early Error Detection**: Issues are caught before they enter the repository
- **Automated Formatting**: No need to manually format code
- **Team Alignment**: Everyone follows the same code standards

### Best Practices
- **Keep rules reasonable**: Overly strict rules can slow down development
- **Document exceptions**: If you need to disable certain rules, document why
- **Use format on save**: Configure your editor to format on save for immediate feedback
- **Regular rule reviews**: Periodically review and update your linting rules

## Troubleshooting

If commits are being blocked unnecessarily:

1. **Check the error messages**: The pre-commit hook will show exactly what's wrong
2. **Fix issues manually**: Address the ESLint errors or warnings
3. **Verify configuration**: Ensure your ESLint and Prettier configurations are compatible
4. **Test lint-staged independently**: Run `npx lint-staged` manually to debug issues

## Conclusion

Setting up pre-commit validation with Husky and lint-staged is a small investment that pays dividends in code quality and team productivity. By automating code formatting and validation, you ensure that your repository maintains high standards without requiring manual intervention from developers.

This setup works particularly well in team environments where maintaining consistent code style is crucial for readability and maintainability. The automated nature of the process means that code quality checks become part of the natural development workflow rather than an additional burden.