+++
title = "5. Maven Release Process and Git Workflow"
weight = 5
date = 2025-11-11
draft = false
tags = ["week1", "JIN"]
+++

## Goal

Learn the proper Maven release workflow to create immutable, versioned releases suitable for production deployments. Master semantic versioning, Git tagging, and the release cycle used by professional development teams.

> **What you'll learn:**
>
> - How to create proper Maven releases (remove SNAPSHOT)
> - When and why to use semantic versioning
> - Git tagging best practices for releases
> - The standard Maven release cycle used in industry
> - How to prepare for the next development iteration

## Prerequisites

> **Before starting, ensure you have:**
>
> - ✓ Completed Tutorial 4: "Configure Systemd Service"
> - ✓ Git installed and project in version control
> - ✓ Understanding of basic Git commands (commit, tag, push)
> - ✓ A working Spring Boot application with SNAPSHOT version

## Exercise Steps

### Overview

1. **Understand SNAPSHOT vs Release Versions**
2. **Prepare and Build Release Version**
3. **Tag Release in Git**
4. **Prepare Next Development Iteration**
5. **Deploy and Verify Release**

### **Step 1:** Understand SNAPSHOT vs Release Versions

Before creating a release, understand the difference between SNAPSHOT and release versions in Maven and why this distinction matters for production deployments.

1. **Check your current version** in `pom.xml`:

   ```bash
   grep "<version>" pom.xml | head -n 1
   ```

   You should see something like `<version>0.0.1-SNAPSHOT</version>`

2. **Examine your target directory:**

   ```bash
   ls -lh target/hellojava-*.jar
   ```

> ℹ **Concept Deep Dive**
>
> **SNAPSHOT versions** indicate work-in-progress code that can change. Maven and CI systems treat SNAPSHOT artifacts as mutable - they can be overwritten with newer builds. This is perfect for development but dangerous for production.
>
> **Release versions** (without SNAPSHOT) are immutable - once published, that version number should never change. This ensures reproducibility: building version 0.0.1 today or in five years produces the exact same artifact. In production environments, you always deploy release versions, never SNAPSHOT versions.
>
> **Semantic Versioning** (MAJOR.MINOR.PATCH):
> - **MAJOR** version: Incompatible API changes (1.0.0 → 2.0.0)
> - **MINOR** version: New functionality, backwards-compatible (1.0.0 → 1.1.0)
> - **PATCH** version: Bug fixes, backwards-compatible (1.0.0 → 1.0.1)
>
> ✓ **Quick check:** You understand why SNAPSHOT versions shouldn't be deployed to production

### **Step 2:** Prepare and Build Release Version

Create a proper release version by removing the SNAPSHOT suffix and building a production-ready artifact. This is the first phase of the Maven release cycle.

1. **Ensure your working directory is clean:**

   ```bash
   git status
   ```

   If there are uncommitted changes, commit them first:

   ```bash
   git add .
   git commit -m "Prepare for first release"
   ```

2. **Update the version** in `pom.xml`:

   **Open** `pom.xml` in your text editor and **locate** the version tag (around line 13):

   ```xml
   <version>0.0.1-SNAPSHOT</version>
   ```

   **Change it to** (remove `-SNAPSHOT`):

   ```xml
   <version>0.0.1</version>
   ```

3. **Save the file** and **build the release version:**

   ```bash
   ./mvnw clean package
   ```

4. **Verify the release JAR was created:**

   ```bash
   ls -lh target/hellojava-0.0.1.jar
   ```

   Note: No `-SNAPSHOT` in the filename

5. **Test the release build:**

   ```bash
   java -jar target/hellojava-0.0.1.jar
   ```

   Wait for "Started HelloJavaApplication" then press `Ctrl+C` to stop

> ℹ **Concept Deep Dive**
>
> Building a clean release (`./mvnw clean package`) ensures no leftover artifacts from previous builds contaminate your release. The `clean` goal removes the entire `target/` directory before building. This is critical for releases to ensure reproducibility.
>
> We test the JAR locally before committing to catch any build issues early. It's much easier to fix problems now than after you've tagged and pushed the release.
>
> ⚠ **Common Mistakes**
>
> - Forgetting to commit before changing versions causes confusion about what code was released
> - Skipping the test run means you might discover the release is broken after tagging
> - Building with tests disabled (`-DskipTests`) might hide breaking changes
> - Not running `clean` can include stale compiled classes
>
> ✓ **Quick check:** JAR file exists at `target/hellojava-0.0.1.jar` (no SNAPSHOT), and application starts successfully

### **Step 3:** Tag Release in Git

Commit the release version and create a Git tag to mark this specific point in history. Tags make it easy to find the exact code that produced a release.

1. **Commit the release version:**

   ```bash
   git add pom.xml
   git commit -m "Release version 0.0.1"
   ```

2. **Create a Git tag:**

   ```bash
   git tag v0.0.1
   ```

3. **Verify the tag was created:**

   ```bash
   git tag
   ```

   Should list: `v0.0.1`

4. **View tag details:**

   ```bash
   git show v0.0.1
   ```

> ℹ **Concept Deep Dive**
>
> **Git tags** mark specific points in your repository's history. Unlike branches, tags don't move - they permanently point to the same commit. This makes them perfect for marking releases.
>
> **Tag naming convention:** Use `v` prefix followed by the version number (v0.0.1, v1.2.3). This is a widely adopted convention that makes it clear you're looking at a version tag.
>
> **Lightweight vs Annotated tags:** We use lightweight tags here (`git tag v0.0.1`), but for important releases, you can create annotated tags with additional metadata:
>
> ```bash
> git tag -a v0.0.1 -m "First production release"
> ```
>
> Annotated tags include the tagger name, email, date, and can have a message. They're treated as full objects in Git and are recommended for production releases.
>
> ⚠ **Common Mistakes**
>
> - Not tagging the release makes it hard to find the exact code for a version later
> - Forgetting the `v` prefix breaks convention and some automated tools
> - Tagging before committing tags the wrong commit (the previous one)
> - Creating tags locally but forgetting to push them to remote
>
> ✓ **Quick check:** `git tag` shows `v0.0.1`, and `git log` shows the release commit

### **Step 4:** Prepare Next Development Iteration

Bump the version to the next SNAPSHOT to prepare for ongoing development. This completes the Maven release cycle.

1. **Open `pom.xml`** again in your text editor

2. **Update the version** to the next SNAPSHOT version:

   **Locate** the version tag:

   ```xml
   <version>0.0.1</version>
   ```

   **Change it to:**

   ```xml
   <version>0.0.2-SNAPSHOT</version>
   ```

3. **Save the file**

4. **Commit the new development version:**

   ```bash
   git add pom.xml
   git commit -m "Prepare for next development iteration (0.0.2-SNAPSHOT)"
   ```

5. **View your recent commits and tags:**

   ```bash
   git log --oneline --decorate -5
   ```

   You should see:
   - HEAD pointing to "Prepare for next development iteration"
   - Tag `v0.0.1` pointing to "Release version 0.0.1"

> ℹ **Concept Deep Dive**
>
> **Why bump immediately?** After releasing 0.0.1, all new commits should be considered "work toward 0.0.2" (or 0.1.0 if you prefer minor version bumps). This prevents the confusion of having multiple commits claiming to be version 0.0.1.
>
> **Version bump strategy:**
> - Patch bump (0.0.1 → 0.0.2): Bug fixes and small improvements
> - Minor bump (0.0.1 → 0.1.0): New features, backwards-compatible
> - Major bump (0.0.1 → 1.0.0): Breaking changes, major milestones
>
> For this educational project, we'll use patch version bumps (0.0.1 → 0.0.2 → 0.0.3) as we add features. In a real project, you'd decide based on the nature of your changes.
>
> **The complete Maven release cycle:**
> 1. Develop features on SNAPSHOT version
> 2. Ready to release? Remove SNAPSHOT, build, test
> 3. Commit release version + create Git tag
> 4. Bump to next SNAPSHOT version
> 5. Push commits and tags to remote
> 6. Deploy release artifact to production
>
> This workflow is used by thousands of Maven projects worldwide and is often automated with Maven Release Plugin or CI/CD pipelines.
>
> ⚠ **Common Mistakes**
>
> - Skipping the bump means the next commit also claims to be 0.0.1
> - Bumping to the wrong version (e.g., 0.0.1 → 0.1.0 for a bug fix)
> - Forgetting to add SNAPSHOT suffix (0.0.2 instead of 0.0.2-SNAPSHOT)
> - Not committing the SNAPSHOT bump separately from the release
>
> ✓ **Quick check:** `pom.xml` shows `0.0.2-SNAPSHOT`, and Git log shows both commits (release and SNAPSHOT bump)

### **Step 5:** Deploy and Verify Release

Deploy the release version to your Azure VM and verify everything works correctly. Then push your Git changes to the remote repository.

1. **Push commits and tags to your repository:**

   ```bash
   git push origin main
   git push origin v0.0.1
   ```

   Or push all tags at once:

   ```bash
   git push origin main --tags
   ```

2. **Deploy the release JAR** to your VM (replace `<YOUR_PUBLIC_IP>` with your VM's IP):

   ```bash
   scp target/hellojava-0.0.1.jar azureuser@<YOUR_PUBLIC_IP>:/tmp/
   ```

3. **SSH into your VM:**

   ```bash
   ssh azureuser@<YOUR_PUBLIC_IP>
   ```

4. **Deploy the release** on the VM:

   ```bash
   sudo systemctl stop hellojava
   sudo mv /tmp/hellojava-0.0.1.jar /opt/hellojava/
   sudo chown hellojava:hellojava /opt/hellojava/hellojava-0.0.1.jar
   sudo chmod 500 /opt/hellojava/hellojava-0.0.1.jar
   sudo ln -sf /opt/hellojava/hellojava-0.0.1.jar /opt/hellojava/hellojava.jar
   sudo systemctl start hellojava
   sudo systemctl status hellojava
   ```

5. **Verify the application** is running:

   Open your browser and navigate to:

   ```
   http://<YOUR_PUBLIC_IP>:8080/
   ```

6. **Check which version is deployed:**

   ```bash
   ls -lh /opt/hellojava/
   ```

   The symlink `hellojava.jar` should point to `hellojava-0.0.1.jar`

> ℹ **Concept Deep Dive**
>
> **Pushing tags separately:** By default, `git push` doesn't push tags. You must explicitly push them with `git push origin <tagname>` or `git push origin --tags` (pushes all tags). This gives you control over what gets published.
>
> **Symlink deployment strategy:** Using a symlink (`hellojava.jar` → `hellojava-0.0.1.jar`) means:
> - systemd service always references `hellojava.jar`
> - To deploy new versions, just update the symlink and restart
> - You can keep multiple versions on disk for easy rollback
> - No need to edit the service file for each deployment
>
> **Version verification:** You can also add version info to your Spring Boot app using `application.properties`:
>
> ```properties
> info.app.version=@project.version@
> ```
>
> Then access it via the `/actuator/info` endpoint (requires Spring Boot Actuator dependency).
>
> ⚠ **Common Mistakes**
>
> - Forgetting to push tags means teammates can't see your releases
> - Deploying the wrong JAR file (SNAPSHOT instead of release)
> - Not testing the deployed application before marking release as complete
> - Pushing tags before verifying the build works
>
> ✓ **Success indicators:**
>
> - Git tag `v0.0.1` visible on GitHub/GitLab
> - Application accessible via public IP
> - Release JAR deployed to `/opt/hellojava/hellojava-0.0.1.jar`
> - Symlink points to correct version
> - No errors in `systemctl status hellojava`

## Test Your Release Process

Verify that your release workflow is correct and your deployment is successful.

1. **Check Git history:**

   ```bash
   git log --oneline --decorate -5
   ```

   Should show:
   - Current HEAD at "Prepare for next development iteration (0.0.2-SNAPSHOT)"
   - Tag `v0.0.1` at "Release version 0.0.1"

2. **Verify current version:**

   ```bash
   grep "<version>" pom.xml | head -n 1
   ```

   Should show: `<version>0.0.2-SNAPSHOT</version>`

3. **Check tag was pushed:**

   ```bash
   git ls-remote --tags origin
   ```

   Should include: `refs/tags/v0.0.1`

4. **Test the deployed application:**

   Visit `http://<YOUR_PUBLIC_IP>:8080/hello?name=Release` in your browser

   Should display: "Hello, Release!"

5. **Verify deployed version on VM:**

   ```bash
   ssh azureuser@<YOUR_PUBLIC_IP> "ls -lh /opt/hellojava/ | grep hellojava"
   ```

   Should show the 0.0.1 JAR and symlink

> ✓ **Success indicators:**
>
> - Git log shows clean release history with proper commit messages
> - Tag `v0.0.1` exists locally and on remote
> - Current working version is `0.0.2-SNAPSHOT`
> - Release JAR (0.0.1) successfully deployed to production VM
> - Application accessible and working correctly
> - Symlink strategy allows easy version management

## Common Issues

> **If you encounter problems:**
>
> **Tag already exists locally:** If you need to recreate a tag, delete it first:
> ```bash
> git tag -d v0.0.1
> ```
>
> **Tag already pushed to remote:** Deleting remote tags requires force:
> ```bash
> git push origin :refs/tags/v0.0.1
> ```
> Then recreate and push again. **Warning:** Only do this for tags that haven't been widely shared.
>
> **Wrong version committed:** Use `git revert` for commits that have been pushed, or `git reset --soft HEAD~1` for local commits to undo the last commit while keeping changes staged.
>
> **Forgot to push tags:** No problem, just push them now:
> ```bash
> git push origin --tags
> ```
>
> **Deployed wrong JAR version:** The symlink makes this easy to fix - just update it:
> ```bash
> sudo ln -sf /opt/hellojava/hellojava-0.0.1.jar /opt/hellojava/hellojava.jar
> sudo systemctl restart hellojava
> ```
>
> **Need to rollback to previous version:** Keep old JARs on the server, then:
> ```bash
> sudo ln -sf /opt/hellojava/hellojava-0.0.0.jar /opt/hellojava/hellojava.jar
> sudo systemctl restart hellojava
> ```

## Summary

You've mastered the Maven release workflow:

- ✓ Understand SNAPSHOT vs release versions
- ✓ Create proper release versions (immutable, versioned)
- ✓ Use Git tags to mark releases
- ✓ Follow the standard Maven release cycle
- ✓ Deploy release artifacts to production
- ✓ Prepare for ongoing development with SNAPSHOT versions

> **Key takeaway:** The Maven release cycle (remove SNAPSHOT → build → tag → bump to next SNAPSHOT) ensures that every release is reproducible and properly versioned. This workflow is the foundation for automated CI/CD pipelines and is used by professional development teams worldwide. Git tags provide traceability, allowing you to find the exact code that produced any release.

## Going Deeper (Optional)

> **Want to explore more?**
>
> - Automate this workflow with Maven Release Plugin
> - Set up automated releases in CI/CD pipelines (GitHub Actions, Azure DevOps)
> - Implement automated version bumping based on commit messages (Conventional Commits)
> - Create release branches for long-term support versions
> - Set up artifact repositories (Nexus, Artifactory) for team sharing
> - Add release notes generation from Git commit history

## Done! 🎉

Excellent work! You now understand professional Maven release workflows and can create properly versioned, immutable releases suitable for production. This knowledge forms the foundation for automated deployments and CI/CD pipelines.

**Next Steps:** In the next tutorial, you'll enhance your deployment security by adding an Nginx reverse proxy in front of your Java application, hiding internal ports and preparing for SSL/TLS certificates.

## Clean Up

No VM cleanup needed - you're using the same VM from Tutorial 4. Your release is now running in production!

To create future releases, simply repeat Steps 2-5 with incremented version numbers.

## Appendix: Automation Script

For future releases, you can automate this workflow with a script. Create `scripts/05-release-with-maven/release.sh`:

```bash
#!/bin/bash
# Automated Maven release script
# Usage: bash scripts/05-release-with-maven/release.sh

set -e  # Exit on any error

# Get current version
CURRENT_VERSION=$(grep "<version>" pom.xml | head -n 1 | sed 's/.*<version>\(.*\)<\/version>/\1/')
RELEASE_VERSION="${CURRENT_VERSION%-SNAPSHOT}"

# Calculate next SNAPSHOT version (increment patch)
IFS='.' read -r major minor patch <<< "$RELEASE_VERSION"
NEXT_VERSION="$major.$minor.$((patch + 1))-SNAPSHOT"

echo "Current version: $CURRENT_VERSION"
echo "Release version: $RELEASE_VERSION"
echo "Next version: $NEXT_VERSION"
echo ""
read -p "Proceed with release? (y/n) " -n 1 -r
echo ""

if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "Release cancelled"
    exit 1
fi

# Ensure clean working directory
if [[ -n $(git status -s) ]]; then
    echo "Error: Working directory is not clean. Commit or stash changes first."
    exit 1
fi

# Remove SNAPSHOT and build
echo "Building release version..."
sed -i.bak "s/<version>$CURRENT_VERSION<\/version>/<version>$RELEASE_VERSION<\/version>/" pom.xml
rm pom.xml.bak
./mvnw clean package

# Commit and tag
git add pom.xml
git commit -m "Release version $RELEASE_VERSION"
git tag "v$RELEASE_VERSION"

# Bump to next SNAPSHOT
echo "Bumping to next development version..."
sed -i.bak "s/<version>$RELEASE_VERSION<\/version>/<version>$NEXT_VERSION<\/version>/" pom.xml
rm pom.xml.bak
git add pom.xml
git commit -m "Prepare for next development iteration ($NEXT_VERSION)"

echo ""
echo "Release complete!"
echo "- Released: $RELEASE_VERSION"
echo "- Tagged: v$RELEASE_VERSION"
echo "- Next: $NEXT_VERSION"
echo ""
echo "Push with: git push origin main --tags"
```

**Usage:**

```bash
# Make it executable
chmod +x scripts/05-release-with-maven/release.sh

# Run it
bash scripts/05-release-with-maven/release.sh

# Push after reviewing
git push origin main --tags
```

This script automates Steps 2-4 of the release process, ensuring consistency and reducing human error.
