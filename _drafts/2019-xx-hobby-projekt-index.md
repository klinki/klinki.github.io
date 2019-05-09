# První appka v ASP.NET Core
1. Začátek
2. Entity framework, databáze, reverse engineering
3. Authentikace, autorizace
4. Digital Ocean, droplet, docker
5. Bitbucket pipeline
6. Konfigurace domény, SSL, nginx

# Zbývá vyřešit:
1. Logování - Serilog
2. Databáze
3. Scheduler - Quartz.NET
4. API dokumentace - swackbuckle, conventions
5. Testování, xUnit, integrační testy

# Hobby projekt v ASP.NET Core - díl první

# Hobby projekt v ASP.NET Core díl pátý - bitbucket pipeline

# Hobby projekt v ASP.NET Core díl šestý - SSL a produkční deployment

# Hobby projekt v ASP.NET Core díl sedmý - databáze
Heslo v `cat /root/.digitalocean_password`

SSH tunnel `ssh root@162.243.1.37 -L 3306:localhost:3306 -N &`
Databáze z docker kontejneru `host.docker.internal`
`ssh -L 3306:localhost:3306 root@68.183.70.45`

`docker exec -it containerid sh`
