# Tools

    # get latest gitversion and nuke global tools
    dotnet tool update GitVersion.Tool --global
    dotnet tool update Nuke.GlobalTool --global


# Git

    # get latest gitversion and nuke global tools
    dotnet tool update GitVersion.Tool --global
    dotnet tool update Nuke.GlobalTool --global
    
    # create git workspace
    git init .
    
    # rename master branch to main
    git branch -m master main
    
    # add a suitable '.gitignore' by hand (see .gitignore-template)
    
    # create directory "_frame"
    mkdir _frame
    
    # add 'gitversion.yml' (see gitversion.yml-template) or create it using the gitversion global tool:
        dotnet-gitversion init
        2) Run getting started wizard
        1) GitFlow (or similar)
        1) Follow SemVer and only increment when a release has been tagged (continuous delivery mode)
        0) Save changes and exit
    
    # initial commit
    git add .gitignore
    git add _frame/gitversion.yml
    git commit -m "initial commit"


# Visual Studio

    # create console application
    cd _frame
    dotnet new console --framework net5.0 --name "<my-project-name>.ConsoleApp"
    
    # then open the project in Visual Studio and save <my-project-name>.sln


# NUKE

    # disable telemetry
    set NUKE_TELEMETRY_OPTOUT=true

    # run nuke global tool (in _frame directory)
    nuke :setup --root .
        🌳 Root directory?    <…>\_frame
        🔩 Build runtime?     .NET (net6.0)
        🔖 Project name?      <my-project-name>.Build
        📍 Project location?  ./<my-project-name>.Build
        💎 Nuke version?      latest
        🧰 Solution?          <my-project-name>.sln
    
    # add reference to GitVersion.Tool
    nuke :add-package GitVersion.Tool

    # add field in Build.cs
    [GitVersion]
    readonly GitVersion GitVersion;
    
    # add to target Compile in Executes
    Serilog.Log.Information($"{GitVersion.FullSemVer}");

    # compile the build project in Visual Studio,
    # then execute the nuke build of the console app from the command line
    nuke
    // prints 0.1.0+<N>

    # the gitversion global tool prints the same version number
    dotnet-gitversion /showvariable FullSemVer

# Workflow

See [https://blog.raulnq.com/semantic-versioning-with-gitversion-gitflow](https://blog.raulnq.com/semantic-versioning-with-gitversion-gitflow).

    # create development branch for 0.x from main
    git checkout -b develop
    {
        git commit -m "starting development 0.x" --allow-empty
        ↪ SemVer = 0.1.0-alpha.<N>

        # create feature branch from development
        git checkout -b feature/<feature-name>
        {
            git commit -m "starting feature" --allow-empty
            ↪ SemVer = 0.1.0-<feature-name>.<N>+<N>

            git commit -m "done with feature" --allow-empty
            ↪ SemVer = 0.1.0-<feature-name>.<N>+<N>
        }

        # merge (and archive|delete) feature branch into develop
        git checkout develop
        git merge --no-ff feature/<feature-name>
        git branch -m feature/<feature-name> feature/×/×<ᵧᵧ.ᵢ><feature-name> // or git branch -d feature/<feature-name>

        git commit -m "done with development" --allow-empty
        ↪ SemVer = 0.1.0-alpha.<N>
    }
    
    # create release branch from development
    git checkout -b release/<mj>.<mi>.<p>
    {
        git commit -m "starting release" --allow-empty
        ↪ SemVer = 1.0.0-beta.<N>+<N>

        git commit -m "done with release preparation → ready for main|production" --allow-empty
        ↪ SemVer = 1.0.0-beta.<N>+<N>
    }

    # merge (and not yet archive) release branch into main
    git checkout main
    git merge --no-ff release/<mj>.<mi>.<p>
    git tag 1.0.0
    ↪ SemVer = 1.0.0
    
    # create development branch for 1.1.x from main
    git checkout -b develop
    {
      # merge (and archive) latest release branch into develop
      git merge --no-ff release/<mj>.<mi>.<p>
      git branch -m release/<mj>.<mi>.<p> release/×/×<mj>.<mi>.<p>

      git commit -m "starting development 1.1.x" --allow-empty
      ↪ SemVer = 2.1.0-alpha.<N>

      ⁞
    }

