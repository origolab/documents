This wiki covers the follow contents about Origo Network:
- [Origo Private Transaction Protocol](Private-Transaction-Protocol)

# Contributing Guidelines


# Rules 

There are a few basic ground-rules for contributors of the project): 



1. No pushing directly to the **develop and stable**branch. 
2. All modifications must be made in a **pull-request**to solicit feedback from other contributors.
3. Pull-requests cannot be merged before **CI runs green** and **at least one reviewer** have given their approval. 
4. Contributors should adhere to the [Ethereum Rust Style Guide](https://wiki.parity.io/Parity-Ethereum-Style-Guide).


### Recommended Practice



1. Create a local copy of the entire project

    _git clone --recurse-submodules [git@github.com](mailto:git@github.com):origolab/medietas.git_



2. Create a feature branch named hot-fix from current develop branch, not modify on the develop branch.

    _git checkout -b hot-fix develop_

3. Make changes and commit

    _git status_


    _git commit -am ‘some comment’_

4. Combine the last two commit into one commit if necessary

    _git rebase -i HEAD~2_


    We can squash the commit and combine the comment in the command line.

5. Local test for project build and CI unit test

    _cargo build --release --features final_


    _./scripts/gitlab/test-linux.sh_

6. Push to the remote hot-fix branch 

    _git push origin hot-fix:hot-fix_

7. Create the pull-request on Github and assign the reviews
8. After the review and the CI runs green, then you can Rebase and merge to the develop branch.
