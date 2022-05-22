# Jackson-Mapper-Examples-

User class example: 

```java
@Getter @Setter
public class User implements Model {

    private final String id;

    @JsonIgnore
    private UUID uuid;
    private ChatColor color;
    private int points;
    private int levelPercentage;
    private int levelInteger;
    private int cahitas;

    private String twitter;
    private String instagram;
    private String twitch;
    private String youtube;

    private boolean socialSpy = false;
    private boolean sounds = true;
    private boolean togglePrivateMessages = true;

    public User(String id) {
        this.id = id;
    }

    @ConstructorProperties({
            "id",
            "uuid",
            "color",
            "points",
            "levelPercentage",
            "levelInteger",
            "cahitas",
            "socialSpy",
            "sounds",
            "togglePrivateMessages",
            "twitter",
            "twitch",
            "instagram",
            "youtube"
    })
    public User(
            String id,
            UUID uuid,
            ChatColor color,
            int points,
            int levelPercentage,
            int levelInteger,
            int cahitas,
            boolean socialSpy,
            boolean sounds,
            boolean togglePrivateMessages,
            String twitter,
            String twitch,
            String instagram,
            String youtube
    ) {
        this(id);
        this.uuid = uuid;
        this.color = color;
        this.points = points;
        this.levelPercentage = levelPercentage;
        this.levelInteger = levelInteger;
        this.cahitas = cahitas;
        this.socialSpy = socialSpy;
        this.sounds = sounds;
        this.togglePrivateMessages = togglePrivateMessages;
        this.twitch = twitch;
        this.twitter = twitter;
        this.instagram = instagram;
        this.youtube = youtube;
    }

    @Override
    public String getId() {
        return id;
    }

    @JsonIgnore
    public String getProgressBar(
            ChatColor completedColor,
            ChatColor notCompletedColor
    ) {
        float percent = (float) this.levelInteger / 100;
        int progressBars = (int) (50 * percent);

        return Strings.repeat("" + completedColor + "|", progressBars)
                + Strings.repeat("" + notCompletedColor + "|", 50 - progressBars);
    }

    @JsonIgnore
    public double getPercentage() {
        int percent = this.levelInteger * 50 / 100;
        double b = Math.round(percent * 10.0) / 10.0;
        return b;
    }
}
```

**User Manager example**

```java
@Getter
public class UserManager {

  private final ObjectRepository<User> userObjectRepository;

  public UserManager(MongoDatabase database) {
    MongoCollection<User> collection = database.getCollection(
            "users",
            User.class
    );
    userObjectRepository = new MongoObjectRepository<>(collection);
  }
}
```

ObjectRepository

```java
public interface ObjectRepository<O extends Model> {

  O find(String id);
  void remove(String id);
  void save(O model);
}
```

MongoObjectRepository

```java
public class MongoObjectRepository<O extends Model> implements ObjectRepository<O> {

  private MongoCollection<O> collection;

  public MongoObjectRepository(
          MongoCollection<O> collection
  ) {
    this.collection = collection;
  }

  @Override
  public O find(String id) {
    return collection.find(Filters.eq("_id", id)).first();
  }

  @Override
  public void remove(String id) {
    this.collection.deleteOne(Filters.eq("_id", id));
  }

  @Override
  public void save(O model) {
    this.collection.replaceOne(
            Filters.eq("_id", model.getId()),
            model,
            new ReplaceOptions().upsert(true)
    );
  }
}
```

Model class

```java
public interface Model {

  @JsonProperty("_id")
  String getId();
}
```


Mongo Connector

```java
@Getter
public class MongoConnector {

  private MongoClient mongoClient;
  private MongoDatabase mongoDatabase;
  private MongoClientURI mongoClientURI;

  public void setup() {
    ObjectMapper objectMapper = this.createObjectMapper();

    CodecRegistry codecRegistry = CodecRegistries.fromRegistries(MongoClient.getDefaultCodecRegistry(),
            CodecRegistries.fromProviders(new JacksonCodecProvider(objectMapper)));
    MongoClientOptions.Builder clientOptions = MongoClientOptions.builder()
            .codecRegistry(codecRegistry);

    this.mongoClientURI = new MongoClientURI(
            "",
            clientOptions
    );

    this.mongoClient = new MongoClient(mongoClientURI);
    this.mongoDatabase = this.mongoClient.getDatabase("");

    Logger mongoLogger = Logger.getLogger("org.mongodb.driver");
    mongoLogger.setLevel(Level.OFF);
  }

  private ObjectMapper createObjectMapper() {
    ObjectMapper mapper = ObjectMapperFactory.createObjectMapper();
    mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
    mapper.registerModule(new Jdk8Module());
    return mapper;
  }
}
```

User listeners


```java
@EventHandler
  public void onAsyncPlayerJoin(AsyncPlayerPreLoginEvent event) {
    String id = event.getUniqueId().toString();
    User user = userManager.getUserObjectRepository().find(id);

    if (user == null) {
      user = new User(id);
    }

    userManager.getUserObjectRepository().save(user);
  }

  @EventHandler
  public void onPlayerQuit(PlayerQuitEvent event) {
    String id = event.getPlayer().getUniqueId().toString();
    User user = userManager.getUserObjectRepository().find(id);

    Bukkit.getScheduler().runTaskAsynchronously(XVideos.getInstance(),
            () ->
                    userManager.getUserObjectRepository().save(user)
    );
  }
  ```


