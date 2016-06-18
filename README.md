# GraphMachine
> GraphQL + Node-Machine

---

### Why

Node-Machines creates typed, modular functions that each have explicit purposes.
GraphMachine will understand the type of data required for the Machines' input and the resulting output, and build a plan for how to take the given information and traverse via Node-Machines to the desired output.

Eventually, each of useful Node-Machines would have this type information associated and the entire community could benefit by having a knowledge graph built from community published Node-Machines.

#### Long-term goal: Knowledge Graph

Strongly typed querying (GraphQL) for artificial intelligence to use to obtain data given some unrelated data.
Such as asking the question `What is @Glavin001's home address?` could recognize that `Glavin001` is the `login` value for `GitHubUser` type and it could traverse through machines of the `GitHubUser` to access the `email` and from `email` could lookup `Contact`s with the same `email`. Finally, the `Contact` has an `address` field.

Thus, the plan is `GitHubUser => Email => Contact => Address`.

Retrieve `email` for `GitHub.login = Glavin001`:

```graphql
{
  github {
    user(login: "Glavin001") {
      email
    }
  }
}
```

Retrieve the `Contact` with that `email`:

```graphql
{
  contacts {
     contact(email: "glavins@email.com") {
       facebook_id
       addresses(location: "home") {
         street
         postal_code
       }
     }
  }
}
```

Notice we could also return the `facebook_id` for the `Contact`. 
Thus, given only a GitHub login/username we could traverse the knowledge graph to obtain unrelated information such as Facebook account, addresses, phone numbers, and much more!


### Input

Add types on top of the Node-Machines.

```javascript
let types = {
  "GitHubUser": {
    "login": "String",
    "email": "String"
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
