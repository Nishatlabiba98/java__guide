# 00 — Setup

Get your machine ready before touching any of the exercises.

## 1. Install a JDK

We're targeting **Java 21 (LTS)**.

Any distribution of OpenJDK works — [Eclipse Temurin](https://adoptium.net/) is a solid default if you don't already have a preference.

- **macOS**: `brew install --cask temurin21`
- **Windows**: download the `.msi` installer from [adoptium.net](https://adoptium.net/) and run it
- **Linux**: use your distro's package manager or [SDKMAN](https://sdkman.io/) if you want to manage multiple JDK versions

Verify the install:

```
java -version
javac -version
```

## 2. Install an IDE

**IntelliJ IDEA** is recommended, especially for interns — it has the best out-of-the-box Java support and catches mistakes early. The free Community Edition is enough for everything in this guide. Download from [jetbrains.com/idea](https://www.jetbrains.com/idea/download/).

When you first open it, make sure the Project SDK is set to the JDK you just installed (`File > Project Structure > SDK`).

## 3. Git basics

You need to be comfortable with this loop before starting the exercises: clone, branch, commit, push.

```
git clone <repo-url>
git checkout -b your-name/00-setup
# make changes
git add <files>
git commit -m "message"
git push -u origin your-name/00-setup
```

Set your identity once, globally, if you haven't already:

```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### SSH

Many workplaces require you to clone over SSH rather than HTTPS, as an added layer of security (no typing passwords/tokens, access is tied to a key on your machine). Set it up once:

```
ssh-keygen -t ed25519 -C "you@example.com"
```

Accept the default file location, then add the generated public key (`~/.ssh/id_ed25519.pub`) to your GitHub/GitLab account under **Settings > SSH Keys**. Test it with:

```
ssh -T git@github.com
```

Once that works, clone using the SSH URL (`git@github.com:org/repo.git`) instead of the HTTPS one.

Section [11 — Git and Code Review](../11-git-and-code-review/README.md) goes deeper into branching and PR etiquette; for now you just need clone/branch/commit/push working end to end.

### Merge conflicts, rebase, stash, and patches

You'll hit these early, so it's worth knowing what they're for before you're stuck mid-conflict:



    
- **Merge conflicts** 
happen when git can't automatically combine changes (usually the same lines edited on two branches). `git status` shows which files are conflicted; open them, look for the `<<<<<<<` / `=======` / `>>>>>>>` markers, edit down to what the code should actually be, then `git add <file>` and continue (`git commit` for a merge, `git rebase --continue` for a rebase).



- **Rebase** 
replays your branch's commits on top of another branch instead of merging the two histories together:

  ```
  git fetch origin
  git rebase origin/main
  ```

  Use this to keep your branch up to date with `main` before opening a PR. Never rebase a branch other people are also working on — it rewrites commit hashes, which breaks their history.



- **Stash** 
temporarily shelves uncommitted(pay attention to UNCOMMITTED) changes so you can switch branches or pull cleanly, then bring them back later:

  ```
  git stash
  # do the other thing (pull, switch branch, etc.)
  git stash pop
  ```

  Useful when someone else's rebase/force-push rewrote the branch history upstream and you need a clean working tree to re-sync before reapplying your work.



- **Patches** 
let you save a set of changes as a file instead of committing/pushing them — handy for handing off work-in-progress or backing it up before a risky rebase:

  ```
  git diff > my-changes.patch     # save uncommitted changes
  git apply my-changes.patch      # reapply them later or elsewhere
  ```

## 4. How to run the exercises

Right now each section is prose + code snippets in its `README.md` — there's no build file yet. To work through a section:

1. Open the section's `README.md`.
2. Create a scratch `.java` file in your IDE (or a `Main.java` in a temp folder) and type out the snippets as you read — don't just read passively.
3. Run it from the IDE (`Shift+F10` in IntelliJ) or from the command line:

   ```
   javac Main.java && java Main
   ```

Some sections may later get a proper Maven/Gradle module with a failing test for you to make pass — if a section has a `pom.xml`, use `mvn test` there instead and follow that section's instructions.

## Exercise: prove the toolchain works

Before moving on, do this once end to end so you know JDK, IDE, and git/SSH are all actually working together:

1. Create `Main.java` with:

   ```java
   public class Main {
       public static void main(String[] args) {
           System.out.println("Hello, TapRoot!");
       }
   }
   ```

2. Run it from the IDE and confirm it prints. Then run it from the command line too:

   ```
   javac Main.java && java Main
   ```

3. Commit and push it on your setup branch:

   ```
   git add Main.java
   git commit -m "00-setup: hello world"
   git push -u origin your-name/00-setup
   ```

If the push succeeds over SSH, your whole toolchain is working — move on to [01 — Types and Control Flow](../01-types-and-control-flow/README.md).

## Checklist

- [ ] `java -version` reports the confirmed version
- [ ] IDE installed and opens without errors
- [ ] SSH key generated and added to your git host
- [ ] Can clone, branch, commit, and push in this repo
- [ ] Comfortable running a single `.java` file from the command line
