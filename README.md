# Boosted Stacks ( yield aggregation for sBTC)

This template should help get you started developing with Vue 3 in Vite.

Smart contracts: 

[Boosted stacks](https://explorer.hiro.so/txid/0xea7d471c16d7edbc48b73b1b407422433448601d2c0a07857533756439f272b4?chain=mainnet)
[Distributor](https://explorer.hiro.so/txid/SP3NBRSFKX28FQ2ZJ1MAKX58HKHSDGNV5N6R5192Q.distributor?chain=mainnet)

# Protocol principle

Alice is a sBTC whale and would like to get 4% APY of rewards right now instead of waiting for 5% APY over a year

Bob is willing to provide the 4% rewards now in exchange of the rewards of the year

Alice is able to get a loan without liquidation risk, because his loan is paid back directly with sBTC rewards from stacks network
Bob won't be able to spend sBTC of Alice, will just get the rewards

You can find the demo of the interface at [Demo](https://main.d1rj3k4uz57trt.amplifyapp.com/)

<img width="959" alt="image" src="https://github.com/user-attachments/assets/b19ffde1-e90b-4137-976a-6c0321b2afc5" />


# Repository how to run it

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Type Support for `.vue` Imports in TS

TypeScript cannot handle type information for `.vue` imports by default, so we replace the `tsc` CLI with `vue-tsc` for type checking. In editors, we need [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) to make the TypeScript language service aware of `.vue` types.

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Type-Check, Compile and Minify for Production

```sh
npm run build
```

### Lint with [ESLint](https://eslint.org/)

```sh
npm run lint
```
