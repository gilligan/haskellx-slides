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
Understanding Nix, learning about the most relevant concepts and tools.

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

---

## Part 1: About

Nix has some very nice qualities

- **reproducibility**: No more _"but it (works for me|worked yesterday)"_
- **reliable**: With atomic upgrades & rollbacks it's hard(er) to screw up
- **functional**: If you hate repetitive YAML files you might like Nix 

---

## Part 1: About

Nix also has some _not-so-nice_ sides

- **complexity**: Only pkg manager w/ the learning curve of Haskell
- **documentation**: Getting better but sometimes still lacking
- **purity**: Colliding with the impure rest of the world

---

## Part 1: About

Some important bits up front

- Nix is available for Linux/macOS
- NixOS is a Linux distro built on top of Nix
- Currently transitioning cli: `nix-*` vs `nix *`
- All packages/libraries/NixOS are maintained in `nixpkgs` (GitHub)
- Packages are available through subscribable **channels**


---

# Enough Talking, Let's start using Nix right away ..

---

## Part 1: Let's Jump Right In!

:computer: **hands-on** :computer:

- Install **cowsay** using **nix-env**: `nix-env -i cowsay`
- Check it's installed using `nix-env -q`
- **Question**: where is cowsay installed to :question:

---

## Part 1: Where are things installed to?

**Q**: Where is cowsay installed to?
**A**: Let's find out ...
```
$ which cowsay
/home/gilligan/.nix-profile/bin/cowsay
```

```shell
$ readlink $(which cowsay)
/nix/store/mhfaj4miflqz1abn60nnkrpaxcg310i9-cowsay-3.03+dfsg2/bin/cowsay
```

`cowsay` sits in the nix store. Let's talk about the **nix store**!

---

## Part 1: Meet the Nix Store

`/nix/store/mhfaj4miflqz1abn60nnkrpaxcg310i9-cowsay-3.03+dfsg2`

- Lives under `/nix/store`
- Violates FHS
- Has funny looking entries

---

## Part 1: Meet the Nix Store

The **Nix Store** is (sort of) the **IO Monad of Nix**

- **Haskell**: You use monadic actions to operate in IO
- **Nix**: You use **derivations** (build actions) to operate on the Store

:arrow_right: So what is a derivation?

---

## Part 1: Derivation ?

A derivation is something that Nix can create for us if we provide at least the following:

- A **name**
- The **system** that we are targeting
- A **builder** command/script


---

## Part 1: Derivation !

Let's try to create the most basic thing:

```haskell
"hello" :: Nix String
```
What would that equate to in Nix?

---

## Part 1: Meet your first Derivation

```nix
# hello.nix
builtins.derivation {
  name = "hello";
  system = "x86_64-linux";
  builder = "/bin/sh";
  args = [ "-c" "echo 'hello' > $out; exit 0"];
}
```

:computer: **hands-on** :computer:
- `$ nix show-derivation $(nix-instantiate ./hello.nix)`
- `$ nix-store --realise $(nix-instantiate ./hello.nix)`

---

## Part 1: Meet your first Derivation

- **.nix**	≅ **.c** : human readable derivation
- **.drv** 	≅ **.o** : machine readable derivation
- **store output path** ≅ **linked executable** : result

also

- `nix-instantiate`: **.nix** :arrow_right: **.drv**
- `nix-store`: **.nix** :arrow_right: **store path**
- `nix-build`: **.nix** :arrow_right: **store path**

---

## Part 1: Are we happy?

```Haskell
"hello" :: Nix String
```

```nix
builtins.derivation {
  name = "hello";
  system = "x86_64-linux";
  builder = "/bin/sh";
  args = [ "-c" "echo 'hello' > $out; exit 0"];
}
```

- `$ nix-build`: gives us a `./result` symlink
- Way too much boilerplate, **we can do better!**

---

## Part 1: Using `<nixpkgs>`

```nix
# default.nix
{pkgs ? import <nixpkgs> {}}:
pkgs.writeText "hello" "hello"
```

Don't worry about the syntax, we'll talk about it shortly...

:computer: **hands-on** :computer:

- `$ cat $(nix-build)`

---

# Nix Language Interlude

---
## Nix Language Interlude

**Attribute Sets**

```nix
{ a = 1; b = 2; c = 3; }
```

```nix
{ a = 1; b = 2; c = 3; }.a # => 1
```

**Recursive Attribute Sets**

```nix
rec { a = 1; b = 2; c = a; }.c # => 1
```

---
## Nix Language Interlude

**Functions**

```nix
let
    add = x: y: x + y
in
    add 40 2 # => 42
```

```nix
let
    add = {x, y}: x + y
in
    add { x = 40; y = 2; } # => 42
```
---

## Nix Language Interlude

**Default Arguments**

```nix
let
    add = {x, y ? 2}: x + y
in
    add { x = 40 } # => 42
```

---

## Nix Language Interlude

**<nixpkgs>**

- `<whatever>` refers to a global variable `whatever`
- `nixpkgs` is defined in the environment variable `NIX_PATH`
- `<nixpkgs>` refers to the currently installed set of nixpkgs

---

# Back To Our Expression ...
---

## Part 1: Using `<nixpkgs>` (again..)

```nix
# default.nix
{pkgs ? import <nixpkgs> {}}:
pkgs.writeText "hello" "hello"
```

- Function with attribute set argument
- Attribute Set with default argument
- Function has `pkgs` in scope and we can make use of nixpkgs
- `nix-env` or `nix-shell` try to call functions when possible

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
