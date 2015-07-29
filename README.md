PagerDuty Dashboard
=============================

[![Dependency Status](https://david-dm.org/gondek/pagerduty-dashboard.svg)](https://david-dm.org/gondek/pagerduty-dashboard)
[![devDependency Status](https://david-dm.org/gondek/pagerduty-dashboard/dev-status.svg)](https://david-dm.org/gondek/pagerduty-dashboard#info=devDependencies)

Grabs services from [PagerDuty](http://www.pagerduty.com/), groups them, and then highlights issues.
For details on the grouping process, refer to the "Conventions" section below.

## Docker Image

Running the [`gondek/pagerduty-dashboard` docker image](https://registry.hub.docker.com/u/gondek/pagerduty-dashboard/) starts the dashboard with the mock data. For deployment, you will have to copy in your API information. You could use a Dockerfile like this (where `config.json` contains the API information):

```
FROM gondek/pagerduty-dashboard
ADD ./config.json /opt/pagerduty-dashboard/
```

## Setup

1. Install [Node.js](https://nodejs.org/) and [Gulp](http://gulpjs.com/)
2. Copy `config.sample.json` to `config.json`, enter your API and app information, and change `mock` to `false`.
3. Install dependencies: `npm install`
4. Build the front-end/client: `gulp`
5. Start the back-end/server: `node app.js`

You can then access the page at `localhost:3000` (or at whatever port was configured).

During development, running `gulp dev` will restart the server and/or run builds when files change.

To use the sample data, set "mock" to `true` to `config.json`.

## Testing

1. Run `npm install -g protractor` to install [`protractor`](https://angular.github.io/protractor/) and `webdriver-manager`
2. Install/update Selenium: `webdriver-manager update`
3. Run Selenium: `webdriver-manager start`
4. Run the server with `mock: true` and `port: 3000` (as in `config.sample.json`)
5. Run `protractor test/protractor.js`

## Custom View Configuration

To configure how the dashboard functions, go to `localhost:3000/#/customize` and note down the generated URL.

## Conventions

These rules determine how the dashboard processes and displays data.

### "Core" vs. "Other" Services

A service that contains `[dashboard-primary]` anywhere in its description is a core service. Core services get separated into core groups. The remaining services get put into the "Other" group.

### "Core" Groups

Core groups are generated from the core services. A colon acts as a separator between the group and service within the service name (eg. `<group>: <service>`). In the interface, these services are labeled as "features". Features with names of `<group>: Site` or `<group>: Server` get separated and enlarged.

Core groups can have dependencies, which are specified in their services. To specify a dependency, add `[dashboard-depends|Some Service,Dependency.*]` to the service's description. Each comma-delimited entry can be a service name (`Some Service`) or a regular expression (`Dependency.*`). Dependencies of dependencies do not get added (i.e. dependencies are only followed to a depth of 1). In the interface, these dependencies are labeled as "services".

A core group's status is only determined from its features (main services) and not its dependencies.

### "Other" Group

If one or more services within the other group are failing, the group gets broken up into two pieces, one holding the offline/failing services and the other holding the online/okay services.

## Ideas

- Play sound on status change (or other event)
- Display assigned users of failing services/features
- Display outage time (either globally or group-wise)
- Dependencies: Allow dependency chains of more than depth 1. Dependency failures trickle up the chain:
  - Immediate (distance 1) dependency failures give the dependent a status of `dependency-down` (a status worse than active and better than warning)
  - Upstream (distance >1) dependency failures give the dependent a status of `dependency-degraded` (a status worse than active and better than `dependency-down`)
  - The new mapping of status numbers would be: ..., active=2, dependency-degraded=3, dependency-down=4, warning=5, ...
  - The two new statuses could have special colors or other ui changes (exclamation or question marks).
- Spin-off project: False-positive analysis and dampen issues from services that tend to fix themselves
