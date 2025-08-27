# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is slock-flexipatch, a fork of slock (simple screen locker) that uses preprocessor directives to conditionally include patches at compile time. The project contains both patched and original code, allowing selective inclusion of patches via configuration.

## Build Commands

- **Quick build and install**: `./install.sh` - Cleans, removes config files, rebuilds, and installs system-wide
- **Clean build**: `make clean && make`
- **Incremental build**: `make`
- **Install**: `sudo make install`
- **Uninstall**: `sudo make uninstall`
- **Note**: No unit tests - this is a screen locker utility

## Repository Structure

### Key Files
- `patches.def.h` - Default patch configuration (copy to `patches.h` before building)
- `config.def.h` - Default slock configuration (copy to `config.h` before building)
- `config.mk` - Build configuration (compiler flags, optional libraries)
- `slock.c` - Main screen locker source with conditional patch code
- `patch/` - Directory containing all patch implementations

### Branch Strategy
- `master` - Pristine copy of upstream (NEVER modify)
- `prod` - Production customizations on top of master
- `test` - Testing branch based on prod

## Architecture

The codebase uses preprocessor directives to conditionally include patches:
- Patches are controlled via `#define PATCH_NAME_PATCH 1` in patches.h
- Code blocks are wrapped in `#if PATCH_NAME_PATCH` directives
- Each patch typically has corresponding .c and .h files in the patch/ directory

### Enabled Patches (from current patches.h)
Currently enabled patches:
- `BLUR_PIXELATED_SCREEN_PATCH` - Pixelated screenshot as lockscreen (pixelation mode active)
- `XRESOURCES_PATCH` - Load colors and settings from Xresources

### Library Dependencies in config.mk
- `IMLIB` - Required for blur/background image patches (currently uncommented)
- `XINERAMA` - Required for message/color message/dwm logo patches (commented out)
- `PAM` - Required for PAM authentication patch (commented out)

## Development Workflow

### Updating from Upstream
1. Ensure upstream remote: `git remote add upstream https://github.com/bakkeby/slock-flexipatch.git`
2. Update master: `git checkout master && git fetch upstream master && git rebase --rebase-merges upstream/master && git push origin master`
3. Rebase prod: `git checkout prod && git rebase --rebase-merges master` (resolve conflicts if any)
4. Rebase test: `git checkout test && git rebase --rebase-merges prod`
5. Build and test: `./install.sh`
6. Push: `git push --force-with-lease origin prod && git push --force-with-lease origin test`

Note: `--rebase-merges` preserves merge commits and avoids re-resolving previous conflicts.

### Adding/Modifying Patches
1. Edit `patches.h` to enable/disable patches
2. Check `config.mk` for required library dependencies
3. Modify patch code in `patch/` directory if needed
4. Follow existing preprocessor directive patterns
5. Test with `./install.sh`

### Quick Reference Commands
- **Check status**: `git status`
- **View recent commits**: `git log --oneline --graph --decorate -10`
- **Compare branches**: `git diff master..prod` or `git diff prod..test`
- **Emergency rollback**: `git checkout prod && git reset --hard origin/prod`

## Code Style
- **Language**: C99 standard
- **Indentation**: Tabs (not spaces)
- **Naming**: snake_case for functions/variables, SCREAMING_SNAKE_CASE for macros
- **Braces**: K&R style
- **Error handling**: Check X11 returns, use `die()` for fatal errors
- **Security**: This is security-critical software - test all changes thoroughly
- **Permissions**: slock requires setuid for screen locking (handled by Makefile)