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

# Enough Talking, Let's Build Something!

---

## Part 2: Haskell Howdy

:computer: **hands-on** :computer:

Let's package (i.e create a derivation for) a program written in Haskell that prints _"howdy"_

---

## Part 2: Haskell Howdy
```
# default.nix
{ pkgs ? import <nixpkgs> {} }:

pkgs.stdenv.mkDerivation {
  pname =                     # :: String
  version =                   # :: String
  src =                       # :: Nix String
  unpackPhase = ":";
  buildInputs =  [ pkgs.ghc ]; 
  buildPhase =                # :: String -- you can reference $src
  installPhase =              # :: String -- place executables in $out/bin
}
```
Use `nix-build` to build, check output in `./result`

---
```
{ pkgs ? import <nixpkgs> {} }:

let
  helloSrc = pkgs.writeText "howdy.hs" ''
    main = putStrLn "howdy"
  '';
in
  pkgs.stdenv.mkDerivation {
    pname = "haskell-howdy";
    version = "0.1.0";
    src = helloSrc;
    unpackPhase = ":";
    buildInputs = with pkgs; [ ghc ];
    buildPhase = ''
      ghc -o howdy $src
    '';
    installPhase = ''
      mkdir -p $out/bin
      cp howdy $out/bin
    '';
  }
```
---

## Part 2: Meet The Shell

There is one very nice tool that we haven't met yet: **nix-shell**

```text
$ nix-shell       # reads shell.nix or default.nix
[nix-shell:~/]$   # shell with all buildInputs
```

It's also possible to specify packages on the command line:

```text
$ nix-shell -p ghc cabal-install
```

---

## Part 2: Creating A Haskell Shell

```nix
# shell.nix

{ pkgs ? import <nixpkgs> {} }:
pkgs.mkShell {
    buildInputs = with pkgs; [ ghc cabal-install ];
    shellHook = ''
        echo ${pkgs.ghc} ${pkgs.cabal-install}
    '';
}
```

- `mkShell` is a convenient wrapper around `mkDerivation`
- `shellHook` is executed when entering the shell

---

# nixpkgs: Programming Languages Integration Interlude

---

## PL Integration Interlude

There is support for integrating different languages/package managers:

- NodeJS (npm2nix, yarn2nix, ...)
- Go (go2nix)
- Rust (carnix, crate2nix, ...)
- Python
- **Haskell**
- ...

---

## PL Integration Interlude

The concept is mostly the same for all languages:

1. Parse dependency specification (`foo.cabal`, `package.json`, ...)
2. Create Nix expressions with a derivation per dependency
3. Provide functions that consume these derivations and invoke compilers/package managers

---

## PL Integration Interlude

**Native workflow**: 
1. Make arbitrary network requests to obtain deps and compile sources

**Nix workflow**: 
1. Populate the Nix Store with dependencies
2. Compile sources using the hashed deps from the Nix store

---

## PL Integration Interlude

- `aeson`, `libpng` and `firefox` are all on the same abstraction level
- Global caching of artifacts for free
- Not always trivial (_cough_, _NodeJS_, _cough_ ...)
- Haskell Integration is among the best

---

## PL Integration Interlude: Haskell

- There is a semi-automatic import of stackage snapshots
- Packages are available under `pkgs.haskellPackages.*`
- `pkgs.haskell.packages."${compiler}".*` for different ghc versions
- Various useful functions are provided (we'll get to those..)
---

## Part 2: Nixifying A Haskell Project

- https://github.com/gilligan/haskellx-code

:computer: **hands-on** :computer:

- Create a `shell.nix` 
- Use `pkgs.mkShell`
- Use `pkgs.haskellPackages.ghcWithPackages (hs: with hs; [<DEPS>])`
- Or use `pkgs.haskell.packages."${compiler}".ghcWithPackages`
- Add argument with default to pick `compiler`

---

## Part 2: Nixifying A Haskell Project

```nix
# shell.nix

{ pkgs ? import <nixpkgs> {}
, ghc ? "ghc865"
}:
let 
  hsPkgs = pkgs.haskell.packages."${ghc}";
  hsEnv = hsPkgs.ghcWithPackages (hsPkgs: with hsPkgs; [ scotty ]);
in
  pkgs.mkShell {
    buildInputs = [ hsEnv pkgs.hlint ];
  }
```
---

## Part 2: Nixifying A Haskell Project

So this works, but now we are redundantly specifying our Haskell dependencies in `shell.nix`. Not cool at all :-1: :-1: :-1:

But no worries, we can do better ...

---

## Part 2: Nixifying A Haskell Project

We can do better by using `cabal2nix`: It parses a `.cabal` and creates an appropriate Nix expression.

:computer: **hands-on** :computer:

```
$ cabal2nix --shell . > shell.nix
$ nix-shell
```
Building the project also works

```
$ nix-build shell.nix
```
---
