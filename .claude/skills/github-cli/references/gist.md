# gh gist

```bash
# Create a gist from a file
gh gist create myfile.go --desc "Example code"

# Create public gist
gh gist create myfile.go --public

# Create from stdin
echo "code snippet" | gh gist create --filename snippet.go

# List gists
gh gist list

# View a gist
gh gist view <gist-id>
```
