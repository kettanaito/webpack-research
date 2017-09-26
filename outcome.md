# Outcome
* **Results:**
  * [Configuration composition](#configuration-composition)
  
## Foreword
For the sake of keeping this document clean, chosen solutions will be marked with :white_check_mark: icon.

## Results
This is the collection of personal findnigs which were acquired during the research:

### Configuration composition
One of the first aspects to consider was a proper approach toward configuration composition. Webpack configurations may easilty get quite big, making them harder to maintain and debug.
Therefore, the following approaches were tested:

#### A. Keeping isolated configurations for different bundles (client, vendor, etc.)
* **Pros:**
  * Less boilerplate, everything needed for the configuration is inside the configuration
  * Easier to debug
* **Cons:**
  * Large configuration files
  * Harder to read
  * Repetitive code
  * Impossible to achieve good maintainability for same loaders
#### B. One default configuration (`default.babel.js`) extended by each target configuration (`client.babel.js`)
* **Pros:**
  * Commonly used parts (loaders, plugins, etc.) are at one place, making them easier to maintain
  * Great for ESLint's `import-webpack-resolver`, where you can reference `default.babel.js` with the included resolvers
* **Cons:**
  * Becomes harder to maintain once introducing webpack configurations for different matters (client, server, api).
  Configurations may share certain parts between each other, while having completely unique parts. In some cases, inheriting certain loaders/plugins by extension
  leads to unnecessary processing, making the builds slower and bundles larger
  
#### C. Reusable *parts*, configurations composed from the parts (like LEGO) :white_check_mark:
* **Pros:**
  * Ability to have independant, reusable and maintainble parts
  * Each part may have relatively complex internal logic without making the whole setup more complex
  * Each part may have its own API, preventing from the duplication of code
  * Parts are interchangable, can be easilty replaced or omitted (better debugging)
  * Parts serve as a single source of truth (better maintainence and debugging)
* **Cons:**
  * More boilerplate
  * Parts should have reasonable internal logic, otherwise they would grow to complex, decreasing their readability
