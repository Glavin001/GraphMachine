# GraphMachine
> GraphQL + Node-Machine

---

### Input

Add types on top of the Node-Machines.

```javascript
let types = {
  "GitHubUser": {
    "login": "String"
  },
  "GitHubRepo": {
    "name": "String",
    "owner": "GitHubUser"
  },
  "GitHubCommit": {
    "sha": "String",
    "message": "String",
    "author": "GitHubUser"
  }
};
let machineType = {
  "github": {
    "listRepos":{
      "inputs": {
        "owner": "GitHubUser.login"
      }
      "exits": {
        "success": ["GitHubRepo"]
      }
    },
    "listRepoCommits": {
      "inputs": {
        "repo": "GitHubRepo.name",
        "owner": "GitHubRepo.owner.name"
      },
      "exits": {
        "success": ["GitHubCommit"]
      }
    }
  }
}
```

Query with GraphQL:

```graphql
{
  github {
    listRepos(owner: "balderdashy") {
      listCommits {
        sha
        message
        author {
          login
        }
      }
    }
  }
}
```

### Output

```javascript
var Github = require('machinepack-github');

// Fetch the list of repos belonging to the specified Github user or organization.
Github.listRepos({
    owner: 'balderdashy',
    limit: 30,
    skip: 0,
}).exec({
    // An unexpected error occurred.
    error: function(err) {

    },
    // OK.
    success: function(repos) {

        return Promise.all(repos.map((repo) => {
            // Fetch activity in a github repo.
            Github.listRepoCommits({
                repo: repo.name,
                owner: repo.owner.login,
            }).exec({
                // An unexpected error occurred.
                error: function(err) {

                },
                // OK.
                success: function(commits) {
                    return commits;
                },
            });
        }));

    },
});
```
