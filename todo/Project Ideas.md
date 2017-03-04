# Project Ideas

## Projects monitoring dashboard
Project monitoring dashboard for maintaining multiple project and their library dependencies.
Initial idea is to manage PHP projects and watch their `composer.json` dependencies. Current version of dependencies
would be checked once a day.
 
In ideal world, all watched project would have coprehensive set of acceptance tests and could be easily deployed to Docker.
Project would allow to spin up docker image with updated libraries and run tests on it. If all tests pass, it could allow some
 one click hot deploy of updated `composer.json`.
 
This idea can be easily used for JavaScript ecosystem as well and possibly also for Java, .NET and other languages 
 with package management. 
