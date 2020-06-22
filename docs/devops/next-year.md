
The general steps required for a new TreeHacks year are:

- Change passwords
- Reset data
- Reconfigure root
- Theme other sites
- Renew Stanford SAML

## Change passwords

Change all passwords for accounts. This way, people from previous years from TreeHacks will not have access to any TreeHacks infrastructure. This reduces risk as only the people who know the credentials will be able to use the credentials. This includes:

- All accounts in [Account Inventory](account-inventory.md)
- The mentor signup token in root
- The password in picontrol (dev and prod)
- Gavel admin passwords (dev and prod)

Additionally, you should remove admin permissions from users who are no longer with TreeHacks and have:

- Ownership status in the GitHub organization
- any access among the IAM users in AWS
- Member of the "admin" group in the treehacks-prod Cognito user pool (for root / all TreeHacks sites)

## Reset data

Only root.treehacks.com supports collecting data from multiple years. If the database is big enough, it might be good to eventually clear it out / archive the data, but you won't need to do anything for now.

Make sure you clear the databases that exist in:

- Gavel
- Meet
- Pi Control

<!-- TODO:
  Should we create a backup of these databases? Or is this data okay to discard?
-->

Additionally, do the following operations on the treehacks-prod Cognito user pool:

- Delete all accounts with a "sponsor" role (this way, next year's sponsors can be provisioned new accounts)
- Remove the "mentor" role from all accounts with a "mentor" role
- Remove the "admin" role from all users who are no longer admins (as mentioned above)
- Grant the "admin" role to users who need to become admins for this year (access all applications, etc.)

## Reconfigure root

Create a new theme for root. Copy the existing theme (in this case `pink_palm_tree.ts` and `pink_palm_tree.scss`) to new files. In these files, you can change the theme colors as needed; however, make sure you update the `hackathon_year` variable and `deadlines` array:

```ts
export default {
    "hackathon_year": 2020,
    "hackathon_date_range": "February 14-16",
    "deadlines": [
        {
            "key": "oos",
            "label": "out-of-state",
            "date": "2019-11-19T07:59:00.000Z",
            "display_date": "November 18, 2019"
        },
        {
            "key": "is",
            "label": "in-state",
            "date": "2019-11-26T07:59:00.000Z",
            "display_date": "November 25, 2019"
        },
        {
            "key": "stanford",
            "label": "Stanford student",
            "date": "2020-02-14T07:59:00.000Z",
            "display_date": "February 13, 2020"
        }
    ],
    "logo": require('./assets/logo.svg'),
    "favicon": require("./assets/favicon.ico"),
    "dashboard_background": require("./assets/combined_circuit.svg")
};
```

In [themes/settings.ts](https://github.com/TreeHacks/root/blob/master/src/themes/settings.ts), change the import statement to import the new theme.

```ts
import customConfig from "./pink_palm_tree";
import defaultConfig from "./default";
import { set } from "lodash";

for (let key in customConfig) {
    set(defaultConfig, key, customConfig[key]);
}

export default defaultConfig;
```

You must also change backend constants at [backend/constants.ts](https://github.com/TreeHacks/root/blob/master/backend/constants.ts). Make sure you change the `HACKATHON_YEAR` variable here, as well as other variables involving bus transportation routes and verticals.

## Theme other sites

Theme all of our other sites. This will involve copying and pasting some CSS, because we don't have a way to store multiple themes on our other sites other than root.

## Updating SAML config

Every year, you will need to update TreeHacks' SPDB SAML configuration so that Stanford sign-in keeps working. This configuration expires on September 18 every year. See [Login](../login.md) for more information on how to do this.