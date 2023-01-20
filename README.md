# widow-analytics

WIDOW Analytics will consist of IronBank approved containers for Matomo. This repository contains
the docker-compose to set up the service.

## Contents

- [Services](#services)
- [Matomo](#matomo)
- [Matomo Architecture](#matomo-architecture)
- [Database Limitations](#database-limitations)
- [Installation](#installation)

## Services

1. Matomo - Open source Google Analytics alternative web analytics platform.
1. OpenTelemetry - an observability framework for cloud native software.
1. Observability back-end like Jaeger or Prometheus

## Matomo

Matomo is a downloadable, Free (GPL licensed) web analytics software platform. It provides detailed reports on your website and its visitors, including the search engines and keywords they used, the language they speak, which pages they like, the files they download and so much more.

Commercial Site: <https://matomo.org/>

IronBank: <https://ironbank.dso.mil/repomap/details;registry1Path=matomo-org%252Fmatomo>

Repo1 Source Code: <https://repo1.dso.mil/dsop/matomo-org/matomo/-/tree/master>

### Matomo Architecture

The Matomo service runs on three containers:

1. Front End - NGINX hosted by IronBank
1. Application - Matomo based on PHP and hosted by IronBank
1. Back End - MariaDB database hosted by IronBank

### Database Limitations

Matomo does not currently support PostGres SQL database and has not plans provide that support in the near future.

"Supporting multiple databases was a long term objective for us, but we have found that this project would require a huge amount of coding, testing, and ongoing complex support… As a small team we need to stay focused and unfortunately we are not planning to add support for other databases in the future (unless for those that are compatible with MySQL/MariaDB). Also over the years we found that Matomo on MariaDB/MySQL can track up to a few billions of hits per month so the platform scales really well as is."

Matomo FAQ on supported databases: <https://matomo.org/faq/how-to-install/faq_55/>

AWS RDS supports MariaDB and is approved up to DoD IL6: <https://aws.amazon.com/compliance/services-in-scope/DoD_CC_SRG/>

Alternative: If MariaDB is not supported by P1, another MySQL database can work.

## Installation

```bash
 docker-compose up --build
```

Once the container is built you need to log into matomo to set up the enviornment,
which will include providing the database information, email address, and site you
want to monitor. You also need to create an account for the administrator of the
site which will be allowed to access the dashboard.

Once the configuration is complete, the setup script removes the port from the site.
So for example localhost:3000 is saved in config.ini.php as localhost only and
needs to be edited to allow you to access the admin console.

### Updating the config.ini.php file for admin console

Copy the `config.ini.php` file from the container and modify it.

```bash
 docker cp matomo-widow-app-1:/var/www/html/config/config.ini.php .
```

Update line 13: `trusted_hosts[] = "localhost"` to `trusted_hosts[] = "localhost:8080"`

```bash
docker cp config.ini.php matomo-widow-app-1:/var/www/html/config/config.ini.php
```

You should now be able to access the mamtomo administration site with the user created
during set up.

### Monitoring WIDOW

Start widow-react and widow-python. In order to track user activity, the matomo script is
required in the `index.html` file.

```html
<!-- Matomo -->
<script>
  var _paq = (window._paq = window._paq || []);
  /* tracker methods like "setCustomDimension" should be called before "trackPageView" */
  _paq.push(["setDownloadExtensions", "docx|doc|ato|aco|txt|pdf|xls|xlsx"]);
  _paq.push(["setCustomUrl", "/" + window.location.hash.substr(1)]);
  _paq.push(["setDocumentTitle", "CustomWIDOWpage"]);
  _paq.push(["trackPageView"]);
  _paq.push(["enableLinkTracking"]);
  _paq.push(["enableHeartBeatTimer"]);
  (function () {
    var u = "//localhost:8080/";
    _paq.push(["setTrackerUrl", u + "matomo.php"]);
    _paq.push(["setSiteId", "1"]);
    var d = document,
      g = d.createElement("script"),
      s = d.getElementsByTagName("script")[0];
    g.async = true;
    g.src = u + "matomo.js";
    s.parentNode.insertBefore(g, s);
  })();
</script>
<!-- End Matomo Code -->
```

### Tracking User Navigation on Single Page Application

In order to track user activity on a single page application,
update the matomo code in index.html to

### Limiting User Data Collection

WIDOW does not want to collect data that will uniquely identify
a user or their geo-location / IP Address. Make sure the following configuration changes are in place.

### React pPlugin for Single Page Application

<https://www.npmjs.com/package/@datapunt/matomo-tracker-react>
