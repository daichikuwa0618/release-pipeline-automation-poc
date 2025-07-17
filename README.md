# Release Pipeline Automation PoC

This repository demonstrates a Proof of Concept (PoC) for automated release pipeline management using GitHub Actions workflows. The automation streamlines branch management for parallel development and release processes.

## Overview

The PoC implements automated workflows to handle:

1. **Version Branch Creation**: Automatically configure `develop-x.y.z` branches for parallel development
2. **Release Branch Management**: Automatically manage branch transitions when release branches are created
3. **Branch Synchronization**: *(Planned)* Automatically merge `develop` changes to appropriate version branches

## Implemented Workflows

### 1. Develop Version Branch Automation

**File**: `.github/workflows/branch-automation-develop-version.yml`

**Trigger**: Creation of branches with pattern `develop-x.y.z`

**Actions**:

- Automatically updates `pubspec.yaml` version to match the branch version
- Creates a commit with the version update
- Creates a Pull Request with detailed information
- Includes reminders to update branch protection rules

**Example**: When you create `develop-1.2.0`, the workflow:

- Updates `pubspec.yaml` version to `1.2.0+<current_build_number>`
- Creates a PR titled "feat: parallel development branch for v1.2.0"
- Provides setup instructions including branch protection rules

### 2. Release Branch Automation

**File**: `.github/workflows/branch-automation-release.yml`

**Trigger**: Creation of branches with pattern `release/x.y.z`

**Actions**:

- Calculates the next minor version (e.g., `release/1.1.0` → looks for `develop-1.2.0`)
- Searches for the corresponding `develop-x.(y+1).z` branch
- Creates a PR to merge the development branch to `develop`
- Facilitates transition from parallel development to main development line

**Example**: When you create `release/1.1.0`, the workflow:

- Looks for `develop-1.2.0` branch
- If found, creates a PR to merge `develop-1.2.0` → `develop`
- Includes detailed context about the transition

## Planned Features

### 3. Develop Branch Synchronization *(Not yet implemented)*

**Specification**:

- **Trigger**: Push to `develop` branch
- **Actions**:
  - Find all `develop-*` branches
  - Merge `develop` changes to the branch with the smallest version number
  - Auto-merge if no conflicts (creates merge commit, not fast-forward)
  - Create PR if conflicts exist
  - Do nothing if no `develop-x.y.z` branches exist

## Branch Strategy

This automation supports a parallel development workflow:

1. **Main Development**: Happens on `develop` branch
2. **Parallel Development**: Use `develop-x.y.z` branches for specific versions
3. **Release Preparation**: Use `release/x.y.z` branches for release candidates
4. **Automatic Transitions**: When releases are created, parallel development automatically merges back to main line

## Usage

### Starting Parallel Development

1. Create a `develop-x.y.z` branch from `develop`
2. Push the branch to trigger automation
3. Review and merge the auto-generated PR
4. Update branch protection rules as prompted
5. Continue development on the new branch

### Releasing a Version

1. Create a `release/x.y.z` branch from the appropriate development branch
2. The automation will create a PR to transition the next version development to `develop`
3. Complete your release process
4. Merge the transition PR to continue development on `develop`

## Workflow Permissions

Both workflows require:

- `contents: write` - To create commits and push changes
- `pull-requests: write` - To create pull requests

## Configuration

### Branch Naming Conventions

- **Development branches**: `develop-x.y.z` (e.g., `develop-1.2.0`)
- **Release branches**: `release/x.y.z` (e.g., `release/1.1.0`)

### Version File

The workflows expect a `pubspec.yaml` file with a `version` field in the format:

```yaml
version: x.y.z+build_number
```

## Benefits

- **Automated Setup**: No manual version updates when creating development branches
- **Consistent Process**: Standardized PR creation with proper documentation
- **Reduced Errors**: Automatic calculation of version numbers and branch relationships
- **Clear Transitions**: Explicit PRs for transitioning between development phases
- **Audit Trail**: All changes tracked through PRs with detailed context

---

This PoC demonstrates how GitHub Actions can significantly streamline release pipeline management while maintaining clear oversight and control over the development process.
