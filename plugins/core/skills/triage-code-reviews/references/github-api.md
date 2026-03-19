# GitHub API Reference — PR Review Comments

## Fetch inline comments
```
gh api repos/{owner}/{repo}/pulls/{pr}/comments
```

## Reply to a comment (dismiss with reason)
Use `in_reply_to` on the PR comments endpoint. The `/replies` sub-endpoint returns 404.
```
gh api "repos/{owner}/{repo}/pulls/{pr}/comments" \
  --method POST \
  --field body="Your dismissal reason" \
  --field in_reply_to={comment_id} \
  --field commit_id="$(git rev-parse HEAD)" \
  --field path="path/to/file" \
  --field line={line_number} \
  --field side="RIGHT"
```
Note: `commit_id` must already be pushed to the PR branch — push first if needed.

## Resolve a thread
The REST API has no resolve endpoint — use GraphQL.

Step 1 — get thread node IDs:
```
gh api graphql -f query='
  query {
    repository(owner: "ORG", name: "REPO") {
      pullRequest(number: PR_NUM) {
        reviewThreads(first: 100) {
          nodes {
            id
            isResolved
            comments(first: 1) { nodes { databaseId body } }
          }
        }
      }
    }
  }
'
```

Step 2 — resolve by thread node ID:
```
gh api graphql -f query='
  mutation {
    resolveReviewThread(input: {threadId: "THREAD_NODE_ID"}) {
      thread { isResolved }
    }
  }
'
```
