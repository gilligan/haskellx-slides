---
marp: true
theme: gaia
---

# Haskell :heart: Nix: From Hello World To Production

Tobias Pflug (tobias.pflug@tweag.io)

---

## About Me

```shell
{
    name = "Tobi";
    employer = "Tweag I/O";
    likes = [ 
        "haskell" 
        "nix" 
        "friendly people"
    ];
}
```

---

## About You

Before we get started i'd like to learn a bit about you

* Who has never used Nix before at all? :hand:
* Who has already packaged something with Nix? :hand:
* Who is already using Nix with Haskell? :hand:

---

## About This Workshop

Some things that are important to me about this workshop

:arrow_right: Getting **lost**? Please let me know **!**
:arrow_right: Getting **bored**? Maybe help those who got lost :sweat_smile: :question:
:arrow_right: Nix is pretty **vast**, my goal is to get you _started_ & _intrigued_

---

## Agenda

* **Part 1** : **Getting To Know Nix**
Understanding Nix, learning about the most relevant concepts and tools.

* **Part 2** : **Haskell & Nix**
We will take some Haskell project, nixify it and discover the Nix/Haskell infrastructure together.

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

- A Lazy, Purely Functional Language (_DSL_)
- A Package Manager

---

## Part 1: About

Nix has some very nice qualities

- **Reproducible builds**: No more _"but it (works for me|worked yesterday)"_
- **Safe to use**: With atomic operations, immutability and rollbacks it's hard to screw up
- **Ability to abstract**: If you hate repetitive YAML files you might like Nix 

---

## Part 1: About

Nix also has some _not-so-nice_ sides

- **Complexity**: Only pkg manager w/ the learning curve of Haskell
- **Documentation**: Getting better but sometimes still lacking
- **Purity**: Colliding with the impure rest of the world

---

## Part 1: About

As a Hasell Developer, why should you care about Nix?

- Cabal Hell? Cabal has improved a lot, not an issue anymore!
- If you don't like Cabal you could still use Stack!

**That's fair but ..**

- With Nix I can also track non-Haskell deps
- Nix gives me **one** abstraction method for "everything"
- Can be used in parallel with Cabal/Stack
---

# Enough Talking, Let's start using Nix

---

## Part 1: Let's Jump Right In!

Let's use Nix as a simple package manager: Let's install **ghc**

:computer: **hands-on** :computer:

- Use `nix-env -qa` to list available packages
- Use `nix-env -i` to install (omit version string)
- Run `nix-env -q` to check it's been installed
- Run `nix-env -e` to remove it again

---

## Part 1: Where are things installed to?

When you were using `nix-env` to install **ghc** where was that being installed to?

```shell
$ readlink $(which ghc)
/nix/store/gvshp9yvc6gql09r3cyryj2zgsnfk6br-ghc-8.6.4/bin/ghc-8.6.4
```

We just discovered the **Nix Store** and we should talk about that a bit

---

## Part 1: Meet the Nix Store

```shell
/nix/store/gvshp9yvc6gql09r3cyryj2zgsnfk6br-ghc-8.6.4/bin/ghc-8.6.4
```

- Lives under `/nix/store`
- Everytying you install ends up there (_No littering_)
- Packages are isolated (_Solves a whole class of problems_)
- Has funny looking paths (_sha256 over all build inputs_)

---

## Part 1: Meet the Nix Store

```shell
/nix/store/gvshp9yvc6gql09r3cyryj2zgsnfk6br-ghc-8.6.4/bin/ghc-8.6.4
```

- **Q**: So how does ghc or anything else end up in the store?
- **A**: By realising a derivation

So let's talk about **Derivations** ...

---

## Part 1: Derivation ?

Let's say we want to package & install some project **foo**. We start by writing `foo.nix` which specifies dependencies, build and install instructions.

* `foo.nix` :arrow_right: _nix-build_ :arrow_right: `/nix/store/*-foo/*`
* `foo.nix` :arrow_right: `/nix/store/*-foo.drv` :arrow_right: `/nix/store/*-foo/*`
* _builtins.derivation_ :arrow_right: _nix-build_ :arrow_right: `/nix/store/*-foo.drv` :arrow_right: `/nix/store/*-foo/*`

---

## Part 1: Derivation ?

Once more: What's A Derivation?

- An **attribute set** that can be created by _builtins.derivation_
- A **file** containing easy-to-parse, machine-readable build info
- A **build action** which can be realised into a Nix Store path


---
## Part 1: Derivation ?

A **derivation** is something that we can create via `builtins.derivation` by providing at least the following:

- A **name**
- The **system** that we are targeting
- A **builder** command/script


---

## Part 1: Derivation !

Let's create the most basic "package": A simple text file

---

## Part 1: Writing your first Derivation

```nix
# default.nix

builtins.derivation {
  name = "hello";                               # what should i call it?
  system = builtins.currentSystem;              # where should it run?
  builder = "/bin/sh";                          # how do i build it?
  args = [ "-c" "echo 'hello' > $out; exit 0"]; # how do i build it?
}
```

:computer: **hands-on** :computer:
- `$ cat $(nix-build)`

---

## Part 1: Are we happy?

```nix
# default.nix

builtins.derivation {
  name = "hello";
  system = "x86_64-linux";
  builder = "/bin/sh";
  args = [ "-c" "echo 'hello' > $out; exit 0"];
}
```

- That seems very low-level!
- **We can do better!** but not with builtin functions only

---

## Part 1: nixpkgs and `<nixpkgs>`

- https://github.com/NixOS/nixpkgs : All libs/packages/NixOS
- **nixpkgs** is distributed through **channels**
- `$ nix-channel --list && nix-channel --update`
- `<nixpkgs>`: global variable refering to your **nixpkgs** copy

**A gentle warning**

Using `<nixpkgs>` means you lose reproducibility guarantees. But we are exploring so this is fine and we can carry on for now ...

---

## Part 1: Writing Some Nix

```nix
# default.nix
{pkgs ? import <nixpkgs> {}}:
pkgs.writeText "hello" "hello"
```

Don't worry about the syntax, we'll talk about it shortly...

:computer: **hands-on** :computer:

- `$ cat $(nix-build)`

---

## Part 1: Writing Some Nix

```nix
# default.nix
{pkgs ? import <nixpkgs> {}}:
pkgs.writeText "hello" "hello"
```

**Functions** in Nix:

```nix
let
    f = a: b: a + b;
    g = {a, b}: a + b;
in
    f 1 1 == g {a = 1; b = 1; } # => true
```

---

## Part 1: Writing Some Nix

```nix
# default.nix
{pkgs ? import <nixpkgs> {}}:
pkgs.writeText "hello" "hello"
```

**Default Arguments** in Nix:
```
let
    f = {a, b ? 1}: a + b;
in
    f {a = 1; b = 1; } == f { a = 1; } # => true
```

---

## Part 1: Writing Some Nix

```nix
# default.nix
{pkgs ? import <nixpkgs> {}}:
pkgs.writeText "hello" "hello"
```

**Importing Files** in Nix:
```
let
    versions = import ./versions.nix;
in
    versions.latest
```

---

## Part 1: Writing Some Nix

```nix
# default.nix
{pkgs ? import <nixpkgs> {}}:
pkgs.writeText "hello" "hello"
```

**Arrays** in Nix:
```nix
let
    a = [ "foo" "bar" ]
    b = [ "baz" ]
in
    a ++ b # => [ "foo" "bar" "baz"
```

---

## Part 1: Writing Some Nix

```nix
# default.nix
{pkgs ? import <nixpkgs> {}}:
pkgs.writeText "hello" "hello"
```

**Strings** in Nix:
```nix
let
    a = ''multi
    line
    string'';
    b = "FTW!"
in
    a + b # => "multi\nline\string\FTW!"
```

---

## Part 1: Writing Some Nix

```nix
# default.nix
{pkgs ? import <nixpkgs> {}}:
pkgs.writeText "hello" "hello"
```

Does the above make sense for everyone now :question:

---

## Part 1: Quick Recap :bulb:

- Nix installs things into the Nix Store
- Realising Derivations puts things in the Store
- Nix has a builtin `derivation` function
- **nixpkgs** contains all official libraries and packages
- **nixpkgs** is distributed through channels
- `<nixpkgs>` is a global variable referring to local nixpkgs
- Commands: _nix-env_, _nix-build_, _nix-channel_

---

# End of Part 1 ;-)

---

# Enough Theory, Let's Build Something!

---

## Part 2: Haskell Howdy

:computer: **hands-on** :computer:

Let's package (i.e create a derivation for) a program written in Haskell that simply prints _"howdy"_

---

## Part 2: Haskell Howdy
```nix
# default.nix
{ pkgs ? import <nixpkgs> {} }:

pkgs.stdenv.mkDerivation {
  pname =                     # :: String
  version =                   # :: String
  src =                       # :: FilePath | Nix String
  unpackPhase = ":";          # :: String -- don't do anything
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

## Part 2: Using `nix-shell` environments

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

## Part 2: Using `nix-shell` environments

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

**Native workflow**: 
Make arbitrary network requests to obtain deps and compile sources

**Nix workflow**: 
1. Populate the Nix Store with dependencies
2. Compile sources using the hashed deps from the Nix Store

---

## PL Integration Interlude

There is support for integrating different languages/package managers:

- NodeJS
- Go
- Rust
- Python
- Haskell
- ...

---

## PL Integration Interlude

Mostly the same setup:

1. Tool for parsing dependencies (`foo.cabal`, `Cargo.toml`, ...)
2. Tool generates Nix expressions/derivations per dependency
3. Functions nixpkgs that consumes Nix expressions & triggers build

---

## PL Integration Interlude: Haskell

What **maintainers** care about:

- https://github.com/commercialhaskell/all-cabal-hashes
- **hackage2nix**: creates a *HUGE* `hackage-packages.nix`
- Based on Stackage snapshot

What **we** care about today:

- _cabal2nix_
- **pkgs.haskellPackages.***

---

# End Of Interlude ;-)

---

## Part 2: Nixifying A Haskell Project (1)

https://github.com/gilligan/haskellx-code

:computer: **hands-on** :computer:

```nix
# shell.nix
{ pkgs ? import <nixpkgs> {}}:
let 
# ?? pkgs.haskellPackages.ghcWithPackages (hs: with hs; [ <deps> ]); ??
in ??
```
- Do you have all dependencies you need? cabal-install?
- Run `nix-shell` and try `cabal configure && cabal build`

---

## Part 2: Nixifying A Haskell Project (1)

This is my solution, yours might look similar:

```nix
# shell.nix

{ pkgs ? import <nixpkgs> {}}:
let 
  hsPkgs = pkgs.haskellPackages;
  hsEnv = hsPkgs.ghcWithPackages (hsPkgs: with hsPkgs; [ scotty ]);
in
  pkgs.mkShell {
    buildInputs = [ hsEnv ];
  }
```

---

## Part 2: Nixifying A Haskell Project (1)

So this works, but now we are redundantly specifying our Haskell dependencies in `shell.nix`. Not cool at all :-1: :-1: :-1:

But no worries, **we can do better**

---

## Part 2: Nixifying A Haskell Project (2)

We can do better by using `cabal2nix`: It parses a `.cabal` file and creates an appropriate Nix expression.

:computer: **hands-on** :computer:

```
$ cabal2nix --shell . > shell.nix
```
- Use `nix-env -i cabal2nix` or `nix-shell -p cabal2nix`
- Try `$ nix-shell`
- Try `$ nix-build shell.nix`

---

## Part 2: Nixifying A Haskell Project (3)

`cabal2nix` is nice but do we really have to run it manually all the time? **Nope**, we can use `callCabal2nix`:

:computer: **hands-on** :computer:

```
{ pkgs ? import <nixpkgs> {} }:
let
  hsPkgs = pkgs.haskellPackages;
in
  hsPkgs.callCabal2nix "hello-service" ./. {}
```
- Save as `default.nix` & try `nix-build`

---

## Part 2: Nixifying A Haskell Project (4)

So far we have been sticking to some default ghc version by using

- `pkgs.haskellPackages`

Different versions are also available via

- `pkgs.haskell.packages.${compiler}`

Where compiler can be one of **ghc865**, **ghc864**, **ghc844** (availability depends on nixpkgs version)

---

## Part 2: Nixifying A Haskell Project (4)

Adjust your most recent `default.nix` such that you can pick the ghc version to use

:computer: **hands-on** :computer:

- Add an argument to your top-level function
- Use `pkgs.haskell.packages.${compiler}`
- Try `nix-shell --argstr compiler ghc864`
- Try `nix-build --argstr compiler ghc864`

---

## Part 2: Nixifying A Haskell Project (4)

This is what I end up with: 

```nix
{ pkgs ? import <nixpkgs> {}
, compiler ? "ghc864"
}:

let
  hsPkgs = pkgs.haskell.packages.${compiler};
in
  hsPkgs.callCabal2nix "hello-service" ./. {}
```

---

## Part 2: Nixifying A Haskell Project (5)

Using `callCabal2nix` we get a derivation which can build our project. The shell we get isn't that great though.

What if we want to add some more tools? 

---

## Part 2: Nixifying A Haskell Project (5)

The ability to extend, compose and modify things is one of Nix' strong suits. We can override:

- function arguments: [<pkg>.override](https://nixos.org/nixpkgs/manual/#sec-pkg-override) 
- derivation attributes: [<pkg>.overrideAttrs](https://nixos.org/nixpkgs/manual/#sec-pkg-override)
- nixpkgs as a whole: [packageOverrides](https://nixos.org/nixpkgs/manual/#sec-modify-via-packageOverrides)

:question: 
Our **default.nix** is fine for building but how could we add tools to our **nix-shell** environment?
:question:

---

## Part 2: Nixifying A Haskell Project (5)

Using `overrideAttrs`:

```
{ pkgs ? import <nixpkgs> {}}:
let
  helloUnchecked = pkgs.hello.overrideAttrs (old: rec {
    doCheck = false;
  });
in
  helloUnchecked
```
- `rec` is for **recursive attribute set**
- `old` is the **original attribute set**

---

## Part 2: Nixifying A Haskell Project (5)

Let's create a `shell.nix` that adds `hlint` and `cabal-install`

:computer: **hands-on** :computer:

- Try importing your **default.nix** from your **shell.nix**
- Try to use **overrideAttrs** to modify what you imported
- Run `nix-shell` and check if it works

---

## Part 2: Nixifying A Haskell Project (5)

This is what my solution looks like:

```nix
{ pkgs ? import <nixpkgs> {}
, compiler ? "ghc864"
}:

(import ./. {inherit compiler;}).overrideAttrs(o: {
  buildInputs = o.buildInputs ++ [pkgs.cabal-install pkgs.hlint]; 
})
```

---

## Part 2: Nixifying A Haskell Project (6)

Let's check what we have covered so far

- We can build it with Nix: :white_check_mark:
- We can build it with different ghc versions: :white_check_mark:
- We can define a development environment: :white_check_mark:
- We can push it to production on k8s: :x:

---

## Part 2: Nixifying A Haskell Project (6)

Nix provides some very nice tooling to create docker images

- no need for the docker daemon
- fully deterministic
- completely declarative
- composable

The output is a tarball that can be loaded via `docker load` or pushed to your registry using tools like [skopeo](https://github.com/containers/skopeo) (no docker daemon needed).

---

## Part 2: Nixifying A Haskell Project (6)

Dockerizing our service is easy as pie: 

```
{ pkgs ? import <nixpkgs> {}
, compiler ? "ghc864"
}:

let 
  hello-service = import ./. { inherit compiler; };
in
  pkgs.dockerTools.buildLayeredImage {
    name = "hello-service";
    contents = [ pkgs.iana-etc ];
    config.Cmd = [ "${hello-service}/bin/hello-service" ];
  }
```

---

## That's All Folks

You are now certified Nix/Haskell Developers :sweat_smile:


---

## By The Way

If you Enjoy **Nix** and **Haskell** and happen to be looking for a job, **we are hiring**:

https://tweag.io

---

## Links

- [Nix By Example](https://medium.com/@MrJamesFisher/nix-by-example-a0063a1a4c55) blog by **James Fisher**
- [A gentle introduction to the Nix family](https://ebzzry.io/en/nix/) - blog by **Rommel Martinez**
- [Nix and Haskell in production](https://github.com/Gabriel439/haskell-nix) by **Gabriel Gonzales**
- [Haskell Infra Section](https://nixos.org/nixpkgs/manual/#users-guide-to-the-haskell-infrastructure) in the **nixpkgs manual**
- [haskell.nix](https://input-output-hk.github.io/haskell.nix/) alternative Haskell/Nix infra by **IOHK**
- [static-haskell-nix](https://github.com/nh2/static-haskell-nix) fully static Haskell builds by **Niklas Hambüchen**

---

## Part 2: Bonus Round: VM Testing 

How about we create an integration test in a single, small Nix file?
