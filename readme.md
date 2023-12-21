# Sample Maven project with conventional commits

This is an sample project that demonstrates how to use the conventional commits standards to simplify Maven versioning steps.

## Basic Tools

- Maven or Maven wrapper
- Git
- Git MkVer



## Get Started

### 1. Install [Git CLI](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) and [Git-MkVer CLI](https://idc101.github.io/git-mkver/installation).
### 2. Install [Maven CLI](https://www.baeldung.com/install-maven-on-windows-linux-mac) or use an embedded [Maven Wrapper CLI](https://www.baeldung.com/maven-wrapper)
### 3. Clone or initialize local git repository.

```bash
git clone git@github.com:alansferreira/git-mkver-with-maven.git 
cd git-mkver-with-maven
```
Or

```bash
mkdir git-mkver-with-maven
git init
```
### 

### 2. Initialize the `.semver` file .

```bash
echo "0.0.0" > .semver
```
### 3. Well done, now you can just make yours commits following the **conventional commits** on your messages and use this commands bellow:

```bash
# Create an 'fix' commit
git commit -m "fix: this is an fix commit" --allow-empty

# Bumping new version
git mkver patch 
# Use Maven CLI to sync 'pom.xml' version with new '.semver' value
./mvnw versions:set -DnewVersion="$(cat .semver)" versions:commit

# Commit Bumped files
git add .semver pom.xml
git commit -m "build: [skip-ci] bumping to $(cat .semver)"
git mkver tag

# Push your(s) tag(s) to remote
git push --follow-tags
```


## Extras
### Enforce check conventional commits with Git Hooks

Put this configs on your **pom.xml** and run maven prepare stages like `./mvnw clean` or `./mvnw install`
```xml
<project>
  <properties>
    ...
    <githook.pre-push>
      # exec ./mvnw build
      # exec ./mvnw test
    </githook.pre-push>
    <githook.commit-msg>
      #!/bin/bash

      COMMIT_MSG=$(cat $1)
      CV_REGEX='^(build|chore|ci|docs|feat|fix|perf|refactor|revert|style|test)(\((\w+)\))?[ ]{0,}(!)?[ ]{0,}\:[ ]{1,}(.+)'
      CV_REGEX_SKIP='^\[maven-release-plugin\]'
      
      [[ $COMMIT_MSG =~ $CV_REGEX ]] &amp;&amp; exit 0
      [[ $COMMIT_MSG =~ $CV_REGEX_SKIP ]] &amp;&amp; exit 0
      
      printf "\033[0;33m'$COMMIT_MSG' message is not permited on conventional commits.\033[0m \n\n"
      
      printf "\033[0;31mWARN: if you do want skip this step, use the '--no-verify' argument on ours 'git' commands.\033[0m \n\n"

      printf "Your message is not conform with Conventional Commits!\n"
      printf "  Please check https://www.conventionalcommits.org/en/v1.0.0/ for messages formats!\n"
      printf "\n"
      printf "  Bellow has some message samples:\n"
      printf "\n"
      printf "    git commit -m \"\033[0;33mfeat: add new custom feature message\033[0m\"\n"
      printf "    \033[0;32m# typing 'feat' will upgrade minor version number. \n"
      printf "       Ex: '0.1.0' -&gt; '0.2.0'\033[0m\n"
      printf "\n"
      printf "    git commit -m \"\033[0;33mfix(security): add new custom fix message\033[0m\"\n"
      printf "    \033[0;32m# typing 'fix' will upgrade patch version number. \n"
      printf "       Ex: '0.1.0' -&gt; '0.1.1'\033[0m\n"
      printf "\n"
      printf "    git commit -m \"\033[0;31mrefactor(security)!: add new custom refactor message\033[0m\"\n"
      printf "    \033[0;32m# The \033[0;31m'!'\033[0;32m cause BREAK CHANGE and upgrade major version number. \n"
      printf "       Ex: '0.1.0' -&gt; '1.0.0'\033[0m\n"
      
      exit 1

      # Color_Off='\033[0m'       # Text Reset
      # # Regular Colors
      # Black='\033[0;30m'        # Black
      # Red='\033[0;31m'          # Red
      # Green='\033[0;32m'        # Green
      # Yellow='\033[0;33m'       # Yellow
      # Blue='\033[0;34m'         # Blue
      # Purple='\033[0;35m'       # Purple
      # Cyan='\033[0;36m'         # Cyan
      # White='\033[0;37m'        # White

    </githook.commit-msg>
    ...
  </properties>

  ...

  <build>
    <plugins>
      ...
      <plugin>
        <groupId>io.github.phillipuniverse</groupId>
        <artifactId>githook-maven-plugin</artifactId>
        <version>1.0.5</version>
        <executions>
          <execution>
            <id>validate:hooks</id>
            <phase>validate</phase>
            <goals>
              <goal>install</goal>
            </goals>
            <configuration>
              <hooks>
                <pre-push>${githook.pre-push}</pre-push>
                <commit-msg>${githook.commit-msg}</commit-msg>
              </hooks>
            </configuration>
          </execution>
          <execution>
            <id>clean:hooks</id>
            <phase>clean</phase>
            <goals>
              <goal>install</goal>
            </goals>
            <configuration>
              <hooks>
                <pre-push>${githook.pre-push}</pre-push>
                <commit-msg>${githook.commit-msg}</commit-msg>
              </hooks>
            </configuration>
          </execution>
        </executions>
      </plugin>
      ...
    </plugins>
  </build>  
</project>
```
