---
sidebar_position: 2
---

# Environment Setup

- This docusaurus notes will correspond to the tutorial from [apollo graphql tutorial series](https://www.apollographql.com/tutorials)

## Node

[Download Latest Version of Node](https://nodejs.org/en/download)

:::tip

- If you already have node installed, but it's outdated, then:

```
brew uninstall node
brew update
brew upgrade
brew cleanup
brew install node
sudo chown -R $(whoami) /usr/local
brew link --overwrite node
brew postinstall node
node -v
npm -v
```

:::

## Clone Repo

```
git clone -b typescript https://github.com/apollographql/odyssey-lift-off-part1
```

## Start the Server App

- The backend app is in the `server/` directory
- Navigate to the repo's `server/` directory and run the following to install dependencies and run the app:

```
npm install && npm run dev
```

## Start the Client App

- The frontend app located in the `client/` directory
- navigate to the repo's `client/` directory and run the following to install dependencies and start the app:

```
npm install && npm start
```
