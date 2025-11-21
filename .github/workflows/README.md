# CI/CD Pipeline Documentation

## Overview
This directory contains GitHub Actions workflows for automating the build and deployment of the HaramBlur Android application.

## Workflows

### android-build.yml
Automatically builds the Android APK on every push and pull request to main/master/develop branches.

**Triggers:**
- Push to main, master, or develop branches
- Pull requests to main, master, or develop branches
- Manual workflow dispatch

**Outputs:**
- Debug APK (always generated)
- Release APK (generated if keystore is available)

**Artifacts:**
- APK files are available for download from the workflow run for 30 days

## Usage

### Downloading APK from Workflow Run

1. Go to the "Actions" tab in your GitHub repository
2. Click on the latest workflow run
3. Scroll down to the "Artifacts" section
4. Download the APK artifact(s)
5. Extract the zip file to get the APK

### Building Release APK with Signing

To build a properly signed release APK, you need to:

1. Add the keystore file to the repository:
   - Place `haramblur-release-key.keystore` in the root directory
   - OR configure GitHub Secrets for secure signing

2. Using GitHub Secrets (recommended for security):
   - Encode your keystore: `base64 haramblur-release-key.keystore > keystore.b64`
   - Add the following secrets in GitHub Settings > Secrets:
     - `KEYSTORE_FILE`: Base64 encoded keystore
     - `KEYSTORE_PASSWORD`: Your keystore password
     - `KEY_ALIAS`: Your key alias
     - `KEY_PASSWORD`: Your key password
   - Update the workflow to decode and use the secrets

### Manual Trigger

You can manually trigger the build:
1. Go to Actions tab
2. Select "Android CI/CD - Build APK"
3. Click "Run workflow"
4. Choose the branch and click "Run workflow"

## Requirements

- JDK 17 (Temurin distribution)
- Gradle 8.13
- Android SDK (automatically installed by GitHub Actions)
- Android Gradle Plugin 8.5.2

## Build Configuration

The build is configured to:
- Use debug signing for builds without a keystore
- Fall back to debug signing for release builds if keystore is missing
- Cache Gradle dependencies to speed up builds
- Run without daemon to work in CI environment
