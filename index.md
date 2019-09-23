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

* **Part 1** : **Getting To Know Nix**
We will get to know Nix, use the cli tools and learn about the most relevant concepts to grok Nix.

* **Part 2** : **Haskell & Nix**
We will take an existing Haskell project, nixify it and make use of the Haskell integration offered by Nix

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
- **Not** a docker rival

---

## Part 1: About

So someone wrote another package manager ...

![wat](https://media.giphy.com/media/8PBfNDoySmsRc49P4F/giphy.gif)

---

## Part 1: About

Nix has some very nice qualities

- deterministic & reproducible (*)
- stateless & declarative
- actually hard to screw up

---

## Part 1: About

Nix also has some not-so-nice sides

- Only package manager with the learning curve of Haskell
- Docs are improving but sometimes lacking
- CLI tools U/X still need some more :heart:
- Colliding with the impure rest of the world

---

## Part 1: Let's Jump Right In!

:computer: **hands-on** :computer:

- Install **ghc** using `nix-env -i`
- Check with `nix-env -q`
- **Where** is ghc installed to? (hint: symlinks)
- **Uninstall** ghc using `nix-env -e`

---

## Part 1: Let's Jump Right In!

So where is ghc installed to exactly

```
$ which ghc
/home/gilligan/.nix-profile/bin/ghc
```

```shell
$ readlink $(which ghc)
/nix/store/gvshp9yvc6gql09r3cyryj2zgsnfk6br-ghc-8.6.4/bin/ghc
```

Let's have a look at `~/.nix-profile` as well ...

---

## Part 1: Let's Jump Right In!

Our user profile directory is symlinked
```shell
$ ls -l ~/.nix-profile
~/.nix-profile -> /nix/var/nix/profiles/per-user/gilligan/profile
```

And our active user profile is also a symlink
```shell
ls -l ~/.nix-profile -> /nix/var/nix/profiles/per-user/gilligan/profile
/nix/var/nix/profiles/per-user/gilligan/profile -> profile-150-link
```

---

## Part 1: Let's Jump Right In!

What have we found out so far 

- `nix-env -i` manipulates the user profile
- Whatever you install ends up in the **nix store**
- Directories in the store all include **hashes**

:arrow_right: **Let's talk about the store ...**

---

## Part 1: Nix Concepts

The **Nix Store** is a central part of Nix

- Your nix store lives at `/nix/store/`
- Everything you install ends up in your nix store
- Packages are installed to `/nix/store/<HASH>-<name>`
- We need _Derivations_ to add something to the store

:arrow_right: **What are Derivations ?**

---

## Part 1: Nix Concepts

A **Derivation** is _build action_

- Takes inputs and when run, creates output(s) in the store
- Can describe a single plain text file or ghc
- All inputs of the derivation determine the associated hash

:arrow_right: **Let's look at a little analogy...**

---

## Part 1: Nix Concepts

- `.nix` ~= `.c` : human readable build description
- `.drv` ~= `.o` : machine readable representation
- `store output path` ~= resulting compiled&linked output

:arrow_right: **Nice but where do the `nix` files come from?**

---

## Part 1: Nix Concepts

You might have heard of **nixpkgs**

- Maintained at https://github.com/NixOS/nixpkgs
- Official nix expression collection
- Also contains all of NixOS

:arrow_right: **Do you have to clone/track this repo? Nope...**

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

:arrow_right: Let's quickly go through the most relevant features

---

## Part 1: Nix Language

Before we begin: If you want to try anything while we go through the
language features, use `nix repl`:

```shell
$ nix repl
Welcome to Nix version 2.2.2. Type :? for help.

nix-repl> :l <nixpkgs>
Added 10091 variables.

nix-repl> 1 + 1
2

nix-repl>
```


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

:computer: **hands-on** :computer:

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

## Part 1: Nix Language

We have already learned about derivations. Nix has a built-in derivation function:

```nix
builtins.derivation { name = "haskellx"; 
                      builder = "/bin/sh"; 
                      system = "x86_64-linux";
                      args = [ "-c" "echo done > $out; exit 0" ];
                    }
```

In practice you usually don't use this, you use `stdenv.mkDerivation` instead :exclamation:

---

## Part 1: Nix Language

**That's it for the language. Any Questions? What's missing :)**

:arrow_right: Next let's move on to talking about tooling

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

`nix-shell` is a great way to create ad-hoc environments!

:computer: **hands-on** :computer:

- Use `nix-shell` to get a shell with `ghc` and `cabal-install`
- Use `-p` to specify packages
- Try running the command with `--pure`, what's different?

:arrow_right: **ad-hoc is nice, declarative is nicer! ...**

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

- Can you mentally parse & undertand the above code? **Questions?**
- Save the above in `shell.nix` & run `nix-shell`

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
