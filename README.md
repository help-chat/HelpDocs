# DocDex
Documentation Index (DocDex) is a utility API which scans javadocs on the web, caches everything into a calculated mixture of memory, file, and database, then provides a JSON API to easily access the information. Warning, don't run this on remote sites, it probably won't work, and you probably won't be able to access that site afterwards.

### Compilation
```bash
./gradlew shadowJar
```
> Archive will be in build/libs/

### Usage
Requirements:
- Java 11
- MongoDB
```bash
java -Xmx2G -jar DocDex.jar
```
Make sure to populate the config.json with your javadocs. On first start, it'll crawl the sites (which can take a while), and then save the indexes to file & database. As mentioned earlier, you should only run this on local sites. Doing otherwise will result in extremely long crawl times, or your ip being banned from said site. Multiple javadoc sites are loaded concurrently to help with speed.

The idea is that after a crawl, the index will be saved into a file (json). This allows the index to easily be moved around. There's two population methods in DocDex, crawl, which scans the web javadocs, and flatfile, which loads from the json file(s). Flatfile is much faster than crawl, so you should always use prebuilt indexes if possible.

Once population has finished, there's two storage methods. DocDex will attempt to save the loaded objects into a flatfile (if one doesn't already exist). It'll then load them into the database (once again if the collection doesn't already exist). The database is where the actual index gets its data from, not the flatfile.

When running the jar, a web server will be spun up on the port & host specified in the config. The route `/index` will then be made available. Two parameters are required to receive a non-null response from this route, javadoc and query. Javadoc refers to one of the names inside your config.json, and the query is one of the objects from the javadoc (e.g. a class or method).

Both methods (methods, constructors) and types (classes, interfaces, enums, annotations) have two identifiers. The first is their name, e.g. `CommandExecutor` (name) or `CommandExecutor#onCommand` (method). Additionally, there's also FQN identifiers in the event of duplicate names, e.g. `org.bukkit.command.CommandExecutor` & `org.bukkit.command.CommandExecutor#onCommand`.

When requesting from the index with a query, it'll first attempt to find a perfect match for the query (two possible matches, name or fqn). If it can't find one, it'll use a ratio of the levenshtein distance of the algorithm, and either the name or fqn (determined by whether the query contains periods) of every object in that javadoc. e.g.
```
/index?javadoc=1.16.4&query=commandexecutor
/index?javadoc=1.16.4&query=commandexecutor~oncommand
```
> Queries are case insensitive.<br/>
> Use ~ instead of # for methods

### Contact
Join either [HelpChat](https://helpch.at/discord), or my personal [discord](https://piggypiglet.me/discord).