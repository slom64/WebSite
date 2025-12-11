
### Handle deletion
```sh
# Stage all deletions
git add -u

# Or stage specific deletions
git add .

# Check status again
git status

# Now commit the deletions
git commit -m "Remove deleted vulnerability files"
```

### repo size
```sh
git count-objects -vH
```