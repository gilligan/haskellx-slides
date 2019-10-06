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

# :coffee:
# How about a little coffee break now?
# :coffee:

---

## Part 2: Quick Recap :bulb:

- Nix installs things into the Nix Store
- Nix Store is like the _IO_ of Nix
- IO : monadic actions ≅ Store : derivations
- Nix has builtin `derivation` function
- _nixpkgs_ provides convenience wrappers
- `nix-env`, `nix-instantiate`, `nix-store`, `nix-build`
- Nix Language (let bindings, functions, args, attrsets) 

---

## Part 2: Another Nix Language Interlude

---

## Part 2: Another Nix Language Interlude

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

## Part 2: Another Nix Language Interlude

Arrays (no separators!)

```nix
[ 4 2 ]  
```

```nix
[ 4 ] ++ [ 2 ]  # => [ 4 2 ]
```
---

## Part 2: Another Nix Language Interlude
## Part 1: Nix Language

Conditionals

```nix
if this == "boring" 
    then "i'm sorry" 
    else "phew"
```

---

## Part 2: Another Nix Language Interlude

The `inherit` keyword injects bindings from the parent scope

```nix
let
    add = {a, b}: a + b;
    a = 40;
    b = 2;
in
    add { inherit a; inherit b; } # == add { a = a; b = b; }
```

---


## Part 2: Another Nix Language Interlude

The `with` keyword brings all attributes from a given set into scope

```nix
let
    fruits = { apples = "A"; oranges = "O"; lemons = "L" };
in
    with foo; [ apples oranges lemons ]
```

---

## Part 2: Another Nix Language Interlude

We can import files passig a literal path to `import`

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

## Part 2: Another Nix Language Interlude

Nasty, useful & not part of the language, `callPackage`:

```nix
# default.nix
with (import <nixpkgs> {}); callPackage foo.nix {}
```

```nix
# foo.nix
{ zlib, gcc }: ...
```

- Looks at expected args (here: _zlib_, _gcc_)
- Automatically passes args with matching name

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
