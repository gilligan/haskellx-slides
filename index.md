---
marp: true
theme: gaia
---

# Haskell :heart: Nix: From Hello World To Production

Tobias Pflug (tobias.pflug@tweag.io)

---

## About Me

```nix
{
    name = "Tobi";
    employer = "Tweag I/O";
    likes = [ 
        "haskell" 
        "nix" 
        "rust" 
        "neovim" 
        "friendly people" 
        "teaching Nix"
    ];
}
```

---

## About You

* Who has never used Nix before at all? :hand:
* Who has already packaged something with Nix? :hand:
* Who is already using Nix with Haskell? :hand:

---

## About This Workshop

:arrow_right: Getting **lost**? Please let me know **!**
:arrow_right: Getting **bored**? Maybe help those who got lost :sweat_smile: :question:
:arrow_right: Nix is pretty **vast**, my goal is to get you _started_ & _intrigued_
:arrow_right: I'd :heart: for this workshop to be **interactive**

---

## Agenda

* **Part 1** : Getting To Know Nix
    - About
    - Concepts
    - Language
    - Tools
* **Part 2** : Haskell & Nix
    - Nixifying a project
    - Common usecase scenarios
    - Creating a docker image

---


## Part 0: No Nix? Get Nix!

:point_right:  Get Nix
```sh
$ curl https://nixos.org/nix/install | sh
```

:point_right: Check installation
```sh
$ nix --version
nix (Nix) 2.2.2
```
---

## Part 1: So What Is Nix?

- A Functional Expression Language
- A Package Manager
- An Operating System (NixOS)
- --Not-- a docker rival

---

## Part 1: About

So someone wrote another package manager ...

![wat](https://media.giphy.com/media/8PBfNDoySmsRc49P4F/giphy.gif)

---

## Part 1: About

Nix has some very nice qualities

- deterministic & reproducible (..\...)
- stateless & declarative
- actually hard to screw up

---

## Part 1: About

Nix also has some not-so-nice qualities

- Only package manager with the learning curve of Haskell
- Docs are improving but sometimes lacking
- CLI tools U/X still need some more :heart:
- Colliding with the impure rest of the world

---

## Part 1: Nix Concepts

We'll have to go through some central aspects of Nix ...

---

## Part 1: Nix Concepts

The **Nix Store** is a central part of Nix

- Your nix store lives at `/nix/store/`
- Everything you install ends up in your nix store
- Packages are isolated in their own directory
- Packages are installed to `/nix/store/<HASH>-<name>`

---

## Part 1: Nix Concepts

A **Derivation** is _build action_

- Takes inputs and when run creates output(s) in the store
- Can describe a single plain text file or ghc
- All inputs of the derivation determine the associated hash

---

## Part 1: Nix Concepts

How do we go from `.nix` file to something in the nix store?

- `.nix` ~= `.c` : human readable build description
- `.drv` ~= `.o` : machine readable representation
- `store output path` ~= resulting compiled&linked output

---

## Part 1: Nix Concepts

You might have heard of **nixpkgs**

- Maintained at https://github.com/NixOS/nixpkgs
- Official nix expression collection
- Also contains all of NixOS

---

## Part 1: Nix Concepts

Packages can be installed from different **channels**

- A channel refers to a commit in a nixpkgs branch
- Different channels have different update strategies
- `nixpkgs-unstable` is updated frequently
- release channels like `nixos-19.03` are more conservative
- https://howoldis.herokuapp.com/

---

# Any burning questions so far?

---


## Part 1: Nix Language

Fortunately the Nix language is quite simple

- Purely functional
- Lazy Evaluation
- Domain Specific

---

## Part 1: Nix Language

Numbers

```nix
42       
4.2
```

Strings

```nix
"42"     

''
forty
two
''
```
---

## Part 1: Nix Language

Arrays (no separators!)

```nix
[ 4 2 ]  
```

Attribute Sets
```nix
{ a = 4; b = 2; }
```

Recursive Attribute Sets

```nix
rec { a = 5; b = a * 2 }
```

---

## Part 1: Nix Language

Let Bindings

```nix
let
 a = 4;
 b = 2;
in
 a * 10 + b
```

No `where` clauses though ;)

---

## Part 1: Nix Language

Functions in Nix are always just anonymous lambda expressions
```nix
name: "howdy there ${name}"
```

Functions with multiple arguments via currying
```nix
name: thing: "howdy there ${name} i heard you like ${thing}"
```

---

## Part 1: Nix Language

We can also use **attribute sets** to pass multiple arguments:

```nix
let 
    f = { name, thing }: "howdy there ${name} i heard you like ${thing}";
in
    f { name = "you"; thing = "me" }
```

---


## Part 1: Nix Language

Unsurprisingly we have `if/then/else`

```nix
if this == "boring" 
    then "i'm sorry" 
    else "phew"
```

---

## Part 1: Nix Language

The `inherit` keyword injects bindings from the parent scope

```nix
let
    add = {a, b}: a + b;
    a = 40;
    b = 2;
in
    add { inherit a; inherit b; } # == { a = a; b = b; }
```

---


## Part 1: Nix Language

The `with` keyword brings all attributes from a given set into scope

```nix
let
    add = {a, b}: a + b;
    foo = {a = 40; b = 2; };
in
    with foo; add { inherit a; inherit b; }
```

---

## Part 1: Nix Language

The `import` keyword lets us allow nix files by specifying a path

```nix
# add.nix
{
    add = a: b: a + b;
}
```
```nix
# default.nix
let
    add = (import ./add.nix).add;
in
    add 40 2
```

---
## Part 1: Nix Language

`nixpkgs` is the collection of all official nix expressions. We can refer to the **system installed nixpkgs** set in a nix expression via `<nixpkgs>`

:question: **audience question** :question: 

:arrow_right: Why can referring to `<nixpkgs>` be problematic?
:arrow_right: How could we avoid that?

---

## Part 1: Nix Language

nixpkgs defines `callPackage` - it **imports** a function specified by a path and passes all expected arguments that can be found in the current scope to it

```
# foo.nix
{ zlib, hello }: ...
```
```
# default.nix
with (import <nixpkgs> {}); pkgs.callPackage ./foo.nix { }
```

---

## Part 1: Nix Tools 

Let's get to know some of the common Nix tools. We have already learned about the concept of channels. We can use the `nix-channel` tool to work with them:

:computer: **hands-on** :computer:

```shell
$ nix-channel --list   # by default probably nixpkgs-unstable
$ nix-channel --update # should produce some output
```

---

## Part 1: Nix Tools 

`nix-env` is the swiss-army-knife of Nix and you can use it to install or remove software. Let's install **ghc** into our **user profile**


:computer: **hands-on** :computer:

- You can use `nix-env -qaP` to list available packages
- Use `nix-env -i` to install
- **Where** is ghc installed to? (Follow the symlinks :grinning:)
- Use `nix-env -e` to uninstall (Is it really gone? From your disk?)

---

## Part 1: Nix Tools 

`nix-shell` is a great way to create ad-hoc environments. We don't even have to install **ghc** at all.

:computer: **hands-on** :computer:

- Use `nix-shell` to get a shell with `ghc` and `cabal-install`
- Use `-p` to specify packages
- Try running the command with `--pure`, what's different?

---

## Part 1: Nix Tools 

```nix
{ pkgs ? import <nixpkgs> {} }:
pkgs.mkShell {
    buildInputs = with pkgs; [ ghc cabal-install ];
    shellHook = ''
        echo ${pkgs.ghc} ${pkgs.cabal-install}
    '';
}
```

:computer: **hands-on** :computer:

- Save the above in `shell.nix`
- What do you expect to see when you run `nix-shell` ?

---

## Part 2: Nix Tools

```nix
{ pkgs ? import <nixpkgs> {}
, ghc ? "ghc864"
}:
let 
  hsPkgs = pkgs.haskell.packages."${ghc}";
  hsEnv = hsPkgs.ghcWithPackages (hsPkgs: with hsPkgs; [ scotty ]);
in
  pkgs.mkShell {
    buildInputs = [ hsEnv pkgs.hlint ];
  }
```

:computer: **hands-on** :computer:

- Try specifying a different ghc version (**hint**: `nix-shell --arg`)
