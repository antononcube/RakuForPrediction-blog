# Thin MCP Client with [Docker MCP Toolkit](https://docs.docker.com/ai/mcp-catalog-and-toolkit/toolkit/)

Anton Antonov   
May 2026

---

## Introduction

This notebook has a complete usage example of a _thin_ Model Context Protocol (MCP) client of the Raku package ["MCP::Client"](https://raku.land/zef:antononcube/MCP::Client), [AAp1].

The MCP server is run in Docker -- see ["Docker MCP Toolkit"](https://docs.docker.com/ai/mcp-catalog-and-toolkit/toolkit/).

"MCP::Client" provides the class `MCP::Client` which creates from MCP server's methods `LLM::Tool` objects that can be used with Raku's Large Language Model (LLM) framework implemented with ["LLM::Functions"](https://raku.land/zef:antononcube/LLM::Functions), ["LLM::Prompts"](https://raku.land/zef:antononcube/LLM::Prompts), ["Text::SubParsers"](https://raku.land/zef:antononcube/Text::SubParsers); see [AA3÷6, AAp1÷3]. 

*Because the client is thin and its implementation concise, it is designed to allow both (i) quick, "on-the-spot" MCP utilization and (ii) easy understanding of MCP principles.*

**Remark:** Similar workflow based with a "simple" Python MCP server is given in the file ["MCP-client-demo.raku"](https://github.com/antononcube/Raku-MCP-Client/blob/main/examples/MCP-client-demo.raku) and corresponding notebook ["Thin-MCP-client-demo.ipynb"](https://github.com/antononcube/Raku-MCP-Client/blob/main/docs/Thin-MCP-client-demo.ipynb).

**Remark:** The Wolfram Language (WL) paclet ["MCPClient"](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/MCPClient/), [AAp2], has the same mission of the (Raku) package "MCP::Client", but WL's "MCPClient" uses a Functional Programming implementation instead of an Object-Oriented Programming one (as "MCP::Client" does.)

---

## Setup

Load the packages used below.

```raku
use LLM::Functions;
use LLM::Tooling;
use LLM::Prompts;
use Text::SubParsers;

use Data::Translators;
use Data::Importers;
use JSON::Fast;

use MCP::Client;
```

---

## MCP Setup and initialization

Get a Docker profile file and show profile's name and identifier:

```raku
my $profile = data-import("https://raw.githubusercontent.com/antononcube/WL-MCPClient-paclet/refs/heads/main/Resources/default_docker_profile.json");
$profile<name id>.raku
```

```
# ("Default Docker Profile", "default-docker-profile")
```

Import the profile (to Docker):

```raku
my $profileFile = $*TMPDIR ~ '/dockerProfile.json';
spurt($profileFile, to-json($profile));

my @cmd = "/usr/local/bin/docker", "mcp", "profile", "import", $profileFile;

my $proc = run @cmd, :out, :err;
$proc.out.slurp(:close).say;
$proc.err.slurp(:close).say;
```

```
# Imported profile default-docker-profile
# 
# 

```

Set the PATH variable (MacOSX in this example):

```raku
for </usr/local/bin /usr/local/share/bin /usr/local/sbin> {
    %*ENV<PATH> = $_ ~ ':' ~ %*ENV<PATH> unless %*ENV<PATH>.contains($_)
}
```


Setup MCP server process creation command elements:

```raku
sink my @cmd = 'docker', 'mcp', 'gateway', 'run', '--profile', $profile<id>
```

Create the MCP client object:

```raku
my Bool:D $echo = False;
my Numeric:D $sleep = 5;
my $client = MCP::Client.new(:$echo, :$sleep);

sink $client.start(@cmd);
```

Initialize the client:

```raku
$client.initialize;
```

```
# True
```

Instead of using the Docker profile ingestion and loading, using Docker's dashboard a default MCP Toolkit profile can be made and the just used the command `docker mcp gateway run`. Here is how such default profile looks like:

![](https://raw.githubusercontent.com/antononcube/Raku-MCP-Client/refs/heads/main/docs/img/Docker-dashboard-Default-MCP-profile.png)

---

## Tools discovery and creation

Get the MCP server tools list:

```raku
my @mcp-tools = |$client.list-tools();
@mcp-tools.elems
```

```
# 75
```

Randomly pick some tools and make a table of their tool records using only names and descriptions:

```raku
#% html
@mcp-tools.pick(5).sort(*<name>)
==> to-html(field-names => <name description>, align => 'left')
```

<table border="1"><thead><tr><th>name</th><th>description</th></tr></thead><tbody><tr><td align=left>browser_fill_form</td><td align=left>Fill multiple form fields</td></tr><tr><td align=left>browser_select_option</td><td align=left>Select an option in a dropdown</td></tr><tr><td align=left>create_repository</td><td align=left>Create a new GitHub repository in your account or specified organization</td></tr><tr><td align=left>get_commit</td><td align=left>Get details for a commit from a GitHub repository</td></tr><tr><td align=left>get_latest_release</td><td align=left>Get the latest release in a GitHub repository</td></tr></tbody></table>

Show tools of the official GitHub MCP server:

```raku
#% html
@mcp-tools.grep(*<description>.contains('GitHub',:i)) 
==> to-html(field-names => <name description>, align => 'left')
```

<table border="1"><thead><tr><th>name</th><th>description</th></tr></thead><tbody><tr><td align=left>add_issue_comment</td><td align=left>Add a comment to a specific issue in a GitHub repository. Use this tool to add comments to pull requests as well (in this case pass pull request number as issue_number), but only if user is not asking specifically to add review comments.</td></tr><tr><td align=left>assign_copilot_to_issue</td><td align=left>Assign Copilot to a specific issue in a GitHub repository.

This tool can help with the following outcomes:
- a Pull Request created with source code changes to resolve the issue


More information can be found at:
- https://docs.github.com/en/copilot/using-github-copilot/using-copilot-coding-agent-to-work-on-tasks/about-assigning-tasks-to-copilot
</td></tr><tr><td align=left>create_branch</td><td align=left>Create a new branch in a GitHub repository</td></tr><tr><td align=left>create_or_update_file</td><td align=left>Create or update a single file in a GitHub repository. 
If updating, you should provide the SHA of the file you want to update. Use this tool to create or update a file in a GitHub repository remotely; do not use it for local file operations.

In order to obtain the SHA of original file version before updating, use the following git command:
git rev-parse &lt;branch&gt;:&lt;path to file&gt;

SHA MUST be provided for existing file updates.
</td></tr><tr><td align=left>create_pull_request</td><td align=left>Create a new pull request in a GitHub repository.</td></tr><tr><td align=left>create_repository</td><td align=left>Create a new GitHub repository in your account or specified organization</td></tr><tr><td align=left>delete_file</td><td align=left>Delete a file from a GitHub repository</td></tr><tr><td align=left>fork_repository</td><td align=left>Fork a GitHub repository to your account or specified organization</td></tr><tr><td align=left>get_commit</td><td align=left>Get details for a commit from a GitHub repository</td></tr><tr><td align=left>get_file_contents</td><td align=left>Get the contents of a file or directory from a GitHub repository</td></tr><tr><td align=left>get_latest_release</td><td align=left>Get the latest release in a GitHub repository</td></tr><tr><td align=left>get_me</td><td align=left>Get details of the authenticated GitHub user. Use this when a request is about the user&#39;s own profile for GitHub. Or when information is missing to build other tool calls.</td></tr><tr><td align=left>get_release_by_tag</td><td align=left>Get a specific release by its tag name in a GitHub repository</td></tr><tr><td align=left>get_tag</td><td align=left>Get details about a specific git tag in a GitHub repository</td></tr><tr><td align=left>issue_read</td><td align=left>Get information about a specific issue in a GitHub repository.</td></tr><tr><td align=left>issue_write</td><td align=left>Create a new or update an existing issue in a GitHub repository.</td></tr><tr><td align=left>list_branches</td><td align=left>List branches in a GitHub repository</td></tr><tr><td align=left>list_commits</td><td align=left>Get list of commits of a branch in a GitHub repository. Returns at least 30 results per page by default, but can return more if specified using the perPage parameter (up to 100).</td></tr><tr><td align=left>list_issues</td><td align=left>List issues in a GitHub repository. For pagination, use the &#39;endCursor&#39; from the previous response&#39;s &#39;pageInfo&#39; in the &#39;after&#39; parameter.</td></tr><tr><td align=left>list_pull_requests</td><td align=left>List pull requests in a GitHub repository. If the user specifies an author, then DO NOT use this tool and use the search_pull_requests tool instead.</td></tr><tr><td align=left>list_releases</td><td align=left>List releases in a GitHub repository</td></tr><tr><td align=left>list_tags</td><td align=left>List git tags in a GitHub repository</td></tr><tr><td align=left>merge_pull_request</td><td align=left>Merge a pull request in a GitHub repository.</td></tr><tr><td align=left>pull_request_read</td><td align=left>Get information on a specific pull request in GitHub repository.</td></tr><tr><td align=left>push_files</td><td align=left>Push multiple files to a GitHub repository in a single commit</td></tr><tr><td align=left>request_copilot_review</td><td align=left>Request a GitHub Copilot code review for a pull request. Use this for automated feedback on pull requests, usually before requesting a human reviewer.</td></tr><tr><td align=left>search_code</td><td align=left>Fast and precise code search across ALL GitHub repositories using GitHub&#39;s native search engine. Best for finding exact symbols, functions, classes, or specific code patterns.</td></tr><tr><td align=left>search_issues</td><td align=left>Search for issues in GitHub repositories using issues search syntax already scoped to is:issue</td></tr><tr><td align=left>search_pull_requests</td><td align=left>Search for pull requests in GitHub repositories using issues search syntax already scoped to is:pr</td></tr><tr><td align=left>search_repositories</td><td align=left>Find GitHub repositories by name, description, readme, topics, or other metadata. Perfect for discovering projects, finding examples, or locating specific repositories across GitHub.</td></tr><tr><td align=left>search_users</td><td align=left>Find GitHub users by username, real name, or other profile information. Useful for locating developers, contributors, or team members.</td></tr><tr><td align=left>sub_issue_write</td><td align=left>Add a sub-issue to a parent issue in a GitHub repository.</td></tr><tr><td align=left>update_pull_request</td><td align=left>Update an existing pull request in a GitHub repository.</td></tr></tbody></table>

Make `LLM::Tool` objects:

```raku
my @tools = @mcp-tools.grep({ $_<description> ~~ /:i GitHub | YouTube/ }).map({ $client.to-llm-tool($_) });

deduce-type(@tools)
```

```
# Vector((Any), 35)
```

Tools without properties:

```raku
.say for |@mcp-tools.grep({ !$_.<inputSchema><properties>  }).map(*<name description inputSchema>)
```

```
# (browser_close Close the page {$schema => https://json-schema.org/draft/2020-12/schema, additionalProperties => False, properties => {}, type => object})
# (browser_navigate_back Go back to the previous page in the history {$schema => https://json-schema.org/draft/2020-12/schema, additionalProperties => False, properties => {}, type => object})
# (get_me Get details of the authenticated GitHub user. Use this when a request is about the user's own profile for GitHub. Or when information is missing to build other tool calls. {properties => {}, type => object})

```

Make a LLM access configuration with the tools:

```raku
my $conf = llm-configuration('ChatGPT', model => 'gpt-5.3-chat-latest', :@tools, :8192max-tokens);
```

```
# LLM::Configuration(:name("chatgpt"), :model("gpt-5.3-chat-latest"), :module("WWW::OpenAI"), :max-tokens(8192))
```

--- 

## LLM invocations

Find the un-accepted GitHub Pull Requests (PRs) -- using the tool "list_pull_requests":

```raku
#% markdown
my $res = llm-synthesize(
    "Which of my -- antononcube -- GitHub pull requests are pending?", 
    e => $conf);
```

Create an LLM function that is assumed to invoke the GitHub MCP server tool "list_branches":

```raku
sink my &fGHB = -> $repo { 
    llm-synthesize([
            "Give the branches of the GitHub repo:",
            $repo, 
            'Separate the features from the bugfixes',
            llm-prompt('NothingElse')('JSON') 
        ],
        e => $conf,
        form => sub-parser('JSON'):drop
    )
}
```

**Remark:** Currently, `llm-function` of ["LLM::Functions"](https://raku.land/zef:antononcube/LLM::Functions/), [AAp3], does not support the use of LLM-tools, hence the "block-with-LLM-synthesis" definition.

Get branches data from a repository:

```raku
sink my $res = &fGHB('bduggan/raku-jupyter-kernel');
```

```raku
#% html
$res».subst(/ [ bugfix | feature ] '/' /) 
==> to-html(align=>'left')
```

<table border="1"><tr><th>bugfixes</th><td align=left><ul><li>trap_sigint</li><li>work-around-tty-issue</li></ul></td></tr><tr><th>features</th><td align=left><ul><li>add_some_messages</li><li>add-workflows</li><li>better_launcher</li><li>determine_version</li><li>implement_comm_info_request</li><li>more_features</li><li>raku-rename</li><li>rename_executable</li><li>update-readme</li></ul></td></tr></table>

Using a tool of a different MCP server:

```raku
# my $url = 'https://www.youtube.com/watch?v=-QtIVv-oz5Y';
# my $res = llm-synthesize(
#     "Get you get the transcript of this URL: $url",
#     e => $conf);
```

----

## Stopping the MCP server process

Kill the MCP client process:

```raku
$client.kill;
```

----

## References

### Articles, blog posts

[AA1] Anton Antonov, ["LLM function calling workflows (Part 1, OpenAI)"](https://rakuforprediction.wordpress.com/2025/06/01/llm-function-calling-workflows-part-1-openai/), (2025), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA2] Anton Antonov, ["LLM function calling workflows (Part 2, Google's Gemini)"](https://rakuforprediction.wordpress.com/2025/06/01/llm-function-calling-workflows-part-2-google-gemini/), (2025), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA3] Anton Antonov, ["LLM function calling workflows (Part 3, Facilitation)"](https://rakuforprediction.wordpress.com/2025/06/08/llm-function-calling-workflows-part-3-facilitation/), (2025), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA4] Anton Antonov, ["LLM function calling workflows (Part 4, Universal specs)"](https://rakuforprediction.wordpress.com/2025/09/28/llm-function-calling-workflows-part-4-universal-specs/), (2025), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

### Packages

[AAp1] Anton Antonov, [MCP::Client, Raku package](https://github.com/antononcube/Raku-MCP-Client), (2026), [GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov, [MCPClient, Wolfram Language paclet](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/MCPClient/), (2026), [Wolfram Language Paclet Repository](https://resources.wolframcloud.com/PacletRepository).

[AAp3] Anton Antonov, [LLM::Functions, Raku package](https://github.com/antononcube/Raku-LLM-Functions), (2023-2026), [GitHub/antononcube](https://github.com/antononcube).

[AAp4] Anton Antonov, [LLM::Prompts, Raku package](https://github.com/antononcube/Raku-LLM-Prompts), (2023-2026), [GitHub/antononcube](https://github.com/antononcube).

[AAp5] Anton Antonov, [Text::SubParsers, Raku package](https://github.com/antononcube/Raku-Text-SubParsers), (2023), [GitHub/antononcube](https://github.com/antononcube).