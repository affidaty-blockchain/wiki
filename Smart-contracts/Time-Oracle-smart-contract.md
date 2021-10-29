Time Oracle
===

## Features

- Provides a trusted source for date/time in the blockchain
- The time is stored in unixtime

## Methods

### `init`

- Allows the account owner to initialize the time Oracle.

  ```json
  args: {
      "name": string,                 // friendly name
      "description": string,          // brief description
      "update_interval": integer,     // time update interval
      "initial_time": integer,        // initial time value
  }
  ```

  The oracle data will be stored in the account with key `config`

  ```json
  "config":{
  "name": string,
  "owner": account-id,        // owner account id
  "description": string,
  "update_interval": integer,
  }
  ```

### `update`

- Allows the owner to update the current time
  The new time must be greater than the older one

  ```json
  args: {
      integer     // Unixtime value
  }
  ```

### `get_time`

- Allows anyone to get the current time in unix format

  ```json
  args: {}
  ```

  Returns the current blockchain time as unix-time, e.g.:

  ```json
  {
      1626254690
  }
  ```

### `get_config`

- Allows anyone to get the Time Oracle configuration

  ```json
  args: {}
  ```

  Returns the Time Oracle configuration, e.g.:

  ```json
  {
    "name": "ProperTime Oracle",
    "owner": "QmYHnEQLdf5h7KYbjFPuHSRk2SPgdXrJWFh5W696HPfq7i",
    "description": "Time oracle from ProperTime.org",
    "update_interval": 3600
  }
  ```
