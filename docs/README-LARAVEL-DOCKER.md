Quick Laravel Docker + bootstrap

Files added under `docker/` and `scripts/bootstrap_project.ps1`.

How to use (option A: Docker)
1. Ensure Docker is installed and running.
2. From `d:\HUH\php\docker` run:

   docker-compose up -d --build

3. Copy `.env.example` to `.env` and update DB_* to point to the Postgres service (host=db, port=5432, database=theater_db, user=postgres, password=secret).
4. Exec into app container to run artisan commands if project files are present:

   docker-compose exec app bash
   composer install
   php artisan key:generate
   php artisan migrate --seed
   npm install
   npm run dev

How to use (option B: Bootstrap locally)
1. Run the bootstrap script which creates a new Laravel project in `theater_full`, copies your scaffolding into it, installs deps and runs migrations/seeds:

   powershell -ExecutionPolicy Bypass -File .\scripts\bootstrap_project.ps1

Notes
- The bootstrap script requires composer, php and npm on your machine.
- The Docker setup mounts the parent folder into containers; adjust volumes to your needs.
- Running the script will download Laravel and dependencies (~100+ MB), ensure you have good network.

If you want, I can run the bootstrap here (it will download packages). Confirm and I'll proceed.
