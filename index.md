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
- **reliability**: With atomic upgrades & rollbacks it's hard to screw up
- **abstraction**: If you hate repetitive YAML files you might like Nix 

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

- Install **ghc** using **nix-env**: `nix-env -i ghc`
- Check it's installed using `nix-env -q`
- Remove it again using `nix-env -e ghc`
- Get a shell environment with ghc: `nix-shell -p ghc`

---

## Part 1: Where are things installed to?

**Q**: Where is ghc installed to?
**A**: Let's find out ...
```
$ which ghc
/home/gilligan/.nix-profile/bin/ghc
```

```shell
$ readlink $(which cowsay)
/nix/store/gvshp9yvc6gql09r3cyryj2zgsnfk6br-ghc-8.6.4/bin/ghc-8.6.4
```

`ghc` sits in the nix store. Let's talk about the **Nix Store**!

---

## Part 1: Meet the Nix Store

- Lives under `/nix/store`
- Isolates packages from reach other
- Has funny looking paths (sha256 of all inputs)

---

## Part 1: Meet the Nix Store

The **Nix Store** is (sort of, but not really) the **IO Monad of Nix**

- **Haskell**: You use monadic actions to operate in IO
- **Nix**: You use **derivations** (build actions) to operate on the Store

So what is a derivation and how do we crate one?

---

## Part 1: Derivation ?

A **derivation** is something that we can create via `builtins.derivation` by providing at least the following:

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

## Part 1: Writing your first Derivation

```nix
# hello.nix
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

## Part 1: Meet your first Derivation

What `nix-build` actually does:

1. Create a .drv file from the .nix file: `nix-instantiate`
1. Run the build creating an output path: `nix-store --realise`

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

- That seems very low-level. **We can do better!**

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

**Nix path entries** in Nix:
```shell
$ nix-instantiate --eval -I x=/tmp/f.nix --expr '<x>' # => /tmp/f.nix
```
**Note**: `<nixpkgs>` has a special meaning in that it refers to the system installed set of nixpkgs.

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
- Nix Store is like the _IO_ of Nix
- IO : monadic actions ≅ Store : derivations
- Nix has a builtin `derivation` function
- nixpkgs provides various wrappers around `derivation`
- `nix-env`, `nix-instantiate`, `nix-store`, `nix-build`

---

# End of Part 1 ;-)

---

# Enough Talking, Let's Build Stuff!

---

## Part 2: Haskell Howdy

:computer: **hands-on** :computer:

Let's package (i.e create a derivation for) a program written in Haskell that prints _"howdy"_

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
2. Compile sources using the hashed deps from the Nix store

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

Mostly the same components to any language integration:

1. Tool for parsing dependencies (`foo.cabal`, `Cargo.toml`, ...)
2. Generating Nix expressions/derivations per dependency
3. Functions consuming the tool output & building/compiling

---

## PL Integration Interlude: Haskell

What **maintainers** care about:

- https://github.com/commercialhaskell/all-cabal-hashes
- **hackage2nix** & **cabal2nix**: create *HUGE* `hackage-packages.nix`

What **we** care about today:

- **pkgs.haskellPackages.***

---

# End Of Interlude ;-)

---

## Part 2: Nixifying A Haskell Project (1)

https://github.com/gilligan/haskellx-hello-world

:computer: **hands-on** :computer:

```nix
# shell.nix
{ pkgs ? import <nixpkgs> {}}:
let 
# ?? pkgs.haskellPackages.ghcWithPackages (hs: [ <deps> ]); ??
in
# ??
```

- Try running `$ nix-shell`

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
- Try `$ nix-shell`
- Try `$ nix-build`

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
- Save as `default.nix` & try `nix-shell` and `nix-build`

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

Let's create a `shell.nix` that adds `hlint` and `ghcid`

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

## Links

- [Nix By Example](https://medium.com/@MrJamesFisher/nix-by-example-a0063a1a4c55) blog by **James Fisher**
- [Nix and Haskell in production](https://github.com/Gabriel439/haskell-nix) by **Gabriel Gonzales**
- [Haskell Infra Section](https://nixos.org/nixpkgs/manual/#users-guide-to-the-haskell-infrastructure) in the **nixpkgs manual**
- [haskell.nix](https://input-output-hk.github.io/haskell.nix/) alternative Haskell/Nix infra by **IOHK**
- [static-haskell-nix](https://github.com/nh2/static-haskell-nix) fully static Haskell builds by **Niklas Hambüchen**
