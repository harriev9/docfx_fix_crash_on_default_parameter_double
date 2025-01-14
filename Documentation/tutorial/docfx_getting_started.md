Getting Started with *DocFX*
===============

## 1. What is *DocFX*

*DocFX* is an API documentation generator for .NET, which currently supports C#, VB and F#.
It generates API reference documentation from triple-slash comments in your source code.
It also allows you to use Markdown files to create additional topics such as tutorials and how-tos, and to customize the generated reference documentation.
*DocFX* builds a static HTML website from your source code and Markdown files, which can be easily hosted on any web server (for example, *github.io*).
Also, *DocFX* provides you the flexibility to customize the layout and style of your website through templates.
If you are interested in creating your own website with your own styles, you can follow [how to create custom template](howto_create_custom_template.md) to create custom templates.

*DocFX* also has the following cool features:

* Integration with your source code. You can click "View Source" on an API to navigate to the source code in GitHub (your source code must be pushed to GitHub).
* Cross-platform support. We have an exe version that runs natively on Windows and with Mono it can also run on Linux and macOS.
* Integration with Visual Studio. You can seamlessly use *DocFX* within Visual Studio.
* Markdown extensions. We introduced *DocFX Flavored Markdown(DFM)* to help you write API documentation. DFM is *100%* compatible with *GitHub Flavored Markdown(GFM)* with some useful extensions, like *file inclusion*, *code snippet*, *cross reference*, and *yaml header*.
For a detailed description about DFM, please refer to [DFM](../spec/docfx_flavored_markdown.md).

> [!Warning]
> **Prerequisites** [Visual Studio 2019](https://www.visualstudio.com/downloads/) is needed for `docfx metadata` msbuild projects. It's not required when generating metadata directly from source code (`.cs`, `.vb`) or assemblies (`.dll`)

## 2. Use *DocFX* as a command-line tool

*Step1.* Install DocFX. Choose from one of the following sources:
* **[Chocolatey](https://chocolatey.org/packages/docfx)**: `choco install docfx -y`.
* **[Homebrew](https://formulae.brew.sh/formula/docfx)** (owned by community): 
   * Intel chip: `brew install docfx`.
   * M1 chip: DocFX doesn't support the new mac M1 chips so you will need to use [Rosetta](https://support.apple.com/en-nz/HT211861) to run it. Follow these steps:
      * [Uninstall Homebrew](https://github.com/homebrew/install#uninstall-homebrew)
      * Re-install brew with the following command: `arch -x86_64 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
      * Install docfx: `arch -x86_64 brew install docfx`
      * Run docfx: `arch -x86_64 docfx path/to/docfx.json --serve`
* **GitHub**: download and unzip `docfx.zip` from https://github.com/dotnet/docfx/releases, extract it to a local folder, and add it to PATH so you can run it anywhere.
* **NuGet**: `nuget install docfx.console`. `docfx.exe` is under folder *docfx.console/tools/*.

*Step2.* Create a sample project
```
docfx init -q
```

This command generates a default project named `docfx_project`.

*Step3.* Build the website
```
docfx docfx_project\docfx.json --serve
```

Now you can view the generated website on http://localhost:8080.

> [!Important]
>
> For macOS/Linux users: Use [Mono](https://www.mono-project.com/) version >= 5.10. Do **NOT** install Debian/Ubuntu-provided packages, since they typically do not provide all the packages required (for example, `msbuild` is missing). Once you have the Mono project's repositories added, the following command line can be used to install the prerequisites on Debian/Ubuntu: `sudo apt-get install mono-runtime mono-devel msbuild`.
>
> For macOS users: **DO NOT** Install Mono from Homebrew, but rather from the [Mono download page](https://www.mono-project.com/download/stable/#download-mac).

## 3. Use *DocFX* integrated with Visual Studio

*Step1.* Create a **Class Library (.NET Framework)** project

*Step2.* Right click on the project and select **Manage NuGet Package**

*Step3.* Search and install the [`docfx.console`](https://www.nuget.org/packages/docfx.console/) NuGet package. It adds itself to the build targets and adds the `docfx.json` configuration file along with other files.

*Step4.* **Build** the project, and a `_site` folder will be generated with the documentation.

> [!NOTE]
> *Possible warning*:
> - *Cache is corrupted*: if your project targets multiple frameworks, you have to indicate one to be the main for the documentation, through the [`TargetFramework` property](https://github.com/dotnet/docfx/issues/1254#issuecomment-294080535) in `docfx.json`:
>
>      "metadata": [
>        {
>          "src": "...",
>          "dest": "...",
>          "properties": {
>            "TargetFramework": <one_of_your_framework>
>          }
>        },
>      ]


## 4. Use *DocFX* with a Build Server

## 4.1 Steps

*DocFX* can be used in a Continuous Integration (CI) environment.

Consider a typical scenario: We want to set up a job on a CI service (e.g. [Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/)) to build documents automatically and publish to a website (e.g. [GitHub Pages](https://pages.github.com/)). We can achieve this with a script in a CI job with the following steps:

1. clone repo
2. `choco install docfx`
3. `docfx {docsetPath}`, while you can change `build.dest` in `docfx.json` to set the output path
4. push all the content in `docsetPath/_site` to `gh-pages` branch

> [!NOTE]
> These steps give an overall impression of what to do. You can adjust them depending on actual scenario.

[docfx-seed](https://github.com/docascode/docfx-seed/blob/master/appveyor.yml) project provides a sample integrating with AppVeyor.

## 4.2 Set branch name

Most build systems do not checkout the branch that is being built, but use a `detached head` for the specific commit.  DocFX needs the branch name to implement the `View Source` link in the API documentation.

Many build systems set an environment variable with the branch name.  DocFX uses the following automatically:

- `APPVEYOR_REPO_BRANCH` - [AppVeyor](https://www.appveyor.com/)
- `BUILD_SOURCEBRANCHNAME` - [Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/)
- `CI_BUILD_REF_NAME` - [GitLab CI](https://about.gitlab.com/gitlab-ci/)
- `Git_Branch` - [TeamCity](https://www.jetbrains.com/teamcity/)
- `GIT_BRANCH` - [Jenkins](https://jenkins.io/)
- `GIT_LOCAL_BRANCH` - [Jenkins](https://jenkins.io/)

Setting the environment variable `DOCFX_SOURCE_BRANCH_NAME` tells DocFX which branch name to use, if the CI service you use is not in the list above.

> [!NOTE]
> *Known issue in AppVeyor*: Currently `platform: Any CPU` in *appveyor.yml* causes `docfx metadata` failure. https://github.com/dotnet/docfx/issues/1078

## 5. A seed project to play with *DocFX*

Here is a seed project: https://github.com/docascode/docfx-seed. It contains:

**/** - The site’s root folder, which contains:  
- **docfx.json** - the configuration file that docfx depends upon.	 
- **index.md** - used to create the **Home** page of the site
- **toc.yml** – renders as the navigation menu bar, shown across the top of each page of the website.  

**/articles** - Several conceptual .md files plus image files under /images. These Markdown files are published under the **Articles** section of the menu bar.  

**/restapi** - REST API Swagger .json files, plus a toc.md that references them which are published under the **REST API** section of the menu bar.

**/specs**  - Markdown "overwrite" files that are used to embellish content under the **API Documentation** menu, which is derived from the source code in /src. These are configured in docfx.json using the `"overwrite": "specs/*.md"` entry.

**/src** - A basic Visual Studio solution with 3 projects, which contain source code that defines the types published under the "API Documentation" section of the menu bar. The left-hand TOC shows the hierarchy of types defines, starting with the namespaces.

After running DocFx on the seed project files, additional folders will be created:

**/\_site** - contains all of the site files generated by DocFx, including the resulting HTML/JSON/JS/Images files required by the web site.

**/obj/api** – contains the .yml files for .NET API classes/enums/interfaces contained under /src, plus a TOC.yml used for the "API Documentation" menu item.

> [!Tip]
> It's good practice to separate files with different types into different folders.

## 6. Q&A

1. Q: How do I quickly reference APIs from other APIs or conceptual files?
   A: Use `@uid` syntax.
2. Q: What is `uid` and where do I find `uid`?
   A: Refer to [Cross Reference](../spec/docfx_flavored_markdown.md#cross-reference) section in [DFM](../spec/docfx_flavored_markdown.md).
3. Q: How do I quickly find `uid` in the website?
   A: In the generated website, hit F12 to view source, and look at the title of an API. You can find `uid` in `data-uid` attribute.
