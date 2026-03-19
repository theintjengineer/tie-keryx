# instructions

- clone the repo
- run `bun i`
- run the CLI to bootstrap the project: `bun packages/keryx/keryx.ts new keryx-app`
  - mind the location, so the bootstrapped and the cloned repo are siblings

```txt
.
├── tie-keryx/
│   ├── packages/
│   │   └── keryx/
│   │       ├── keryx.ts
│   │       ├── node_modules
│   │       └── ...
│   ├── package.json
│   └── ...
└── keryx-app/
    ├── actions/
    │   └── ...
    └── schema/
        ├── users.ts
        └── ...
```

- still in the cloned repo, `cd` into the package [`cd packages/keryx`] and do `bun link`
- go to the bootstrapped `keryx-app` dir
- run `bun link keryx`
- create the database and cache containers
  - there are 2 `.yaml` examples here; replace with your `$USER`; feel free to use Docker instead
  - if you have `podman` installed, just edit the `yaml` files to match your Linux `$USER`, and run `podman kube play pg.pod.yaml` and `podman kube play valkey.pod.yaml`
- containers up and running, it's time to run `bun dev` in the **boostrapped** app: `bun dev`
- there shouldn't be any errors
- you should be able to check the integration now; keep going!

#### **creating a user**

```sh
curl -X PUT http://localhost:8080/api/user \
    -H "Content-Type: application/json" \
    -d '{"name":"Jane","email":"jane@test.com","password":"password123"}'
```

#### **creating a session**

- [mind the response with the session id or save it to a file with `-c cookies.txt` (you'll need the UUID for the next step)]

```sh
curl -X PUT http://localhost:8080/api/session \
    -H "Content-Type: application/json" \
    -d '{"email":"jane@test.com","password":"password123"}'
```

#### **accessing the session data**

- use the session `id` returned from the previous step

```sh
curl -b "__session=your-uuid-here" \
    -X GET http://localhost:8080/api/me \
    -H "Content-Type: application/json"
```

#### **deleting the session**

```sh
curl -b "__session=your-uuid-here" -X DELETE http://localhost:8080/api/session
```

#### **ensuring the session's really gone**

```sh
curl -b "__session=your-uuid-here" \
    -X GET http://localhost:8080/api/me \
    -H "Content-Type: application/json"
```

# remarks

- this is a dumm version of actionhero/keryx
- I just wanted to try
  - using Drizzle's beta version,
    - see the new migration files with the `timestamp_random-name` migrations folder, and the `snapshot.json` file
    - see the `packages/keryx/util/scafold.ts`
    - see the `migration.sql.mustache` and `snapshot.json.mustache` files added to the scaffold templates directory11
  - getting rid of the `drizzle-zod` standalone package, since it's now in `drizzle-orm/zod`
  - trying Bun's native SQL driver
- I didn't run the tests since I don't understand them well enough yet to map it down to check the changes
  - also, I think they cover websockets, the project bootstrapping tailored to the specific files the original project uses, the `backend` and `frontend` apps the original app demoes, and so on; so, not appropriate to test this dumbed down version
- in the `db` initialiser, I have

```ts
try {
  const migrationsFolder = path.join(api.rootDir, "drizzle");
  const snapshotPath = path.join(
    migrationsFolder,
    "20260319134928_slow_blue_shield",
    "snapshot.json",
  );
```

which works for the example|scaffolded project, and is also how Drizzle currently spits migration files out, but we gotta make that dynamic, somehow, instead of hardcoding it that way; I just haven't understood `keryx` enough yet to see how this is all linked together|with the other packages componets, flows, etc. so I could try modifying it

- TL;DR: to see the changes, scan the package for
  - bun-sql
  - 20260319134928
  - snapshot.json
  - 1.0.0-beta.18-7eb39f0

# extra

- don't mind the `biome` stuff; I'm working on learning how to use it to format the code, lint, etc. haha
