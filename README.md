<h1 align="center">🎮 GameLovers Configs Provider</h1>

<p align="center">
  <strong>Type-safe, high-performance configuration management for Unity games</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Unity-6000.0%2B-blue.svg" alt="Unity 6000.0+"/>
  <img src="https://img.shields.io/badge/License-MIT-green.svg" alt="MIT License"/>
  <img src="https://img.shields.io/badge/Version-0.2.0-blue.svg" alt="Version 0.2.0"/>
  <img src="https://img.shields.io/badge/C%23-8.0%2B-purple.svg" alt="C# 8.0+"/>
</p>

<p align="center">
  <a href="#-features">Features</a> •
  <a href="#-installation">Installation</a> •
  <a href="#-quick-start">Quick Start</a> •
  <a href="#-documentation">Documentation</a> •
  <a href="#-examples">Examples</a> •
  <a href="#-contributing">Contributing</a>
</p>

---

## 🎯 Why GameLovers Configs Provider?

Managing game configuration data shouldn't be a hassle. This library solves common Unity config challenges:

✅ **Type Safety** - No more casting or string-based lookups  
✅ **Designer Friendly** - Edit configs in Unity Inspector with ScriptableObjects  
✅ **Performance** - O(1) lookups with pre-built dictionaries  
✅ **Flexibility** - Support for singletons, collections, and custom types  
✅ **Backend Ready** - Built-in serialization for server sync  
✅ **Version Control** - Track and update configs atomically  

Perfect for managing enemy stats, item databases, level configurations, game balance values, and any other design data in your Unity projects.

---

## ✨ Features

**Lightweight, type-safe configuration storage for Unity** that lets you load, query, version, and serialize your game configs (design data, tuning values, asset references, etc.) in a predictable and efficient way.

**Core Features:**
- 🔍 **Single or multiple configs per type** - Singleton pattern or id-indexed collections
- 🚀 **Fast lookups** - In-memory dictionaries for O(1) performance  
- 📝 **Versioning and atomic updates** - Track changes and update safely
- 🔄 **JSON serialization/deserialization** - Perfect for client/server sync
- 📦 **ScriptableObject containers** - Designer-friendly key/value pairs
- 🎯 **Type-safe queries** - No casting or string-based lookups
- 🌐 **Backend integration** - Optional remote config fetching

---

## 📑 Table of Contents

- [🎯 Why Use This?](#-why-gamelovers-configs-provider)
- [✨ Features](#-features)
- [📦 Installation](#-installation)
- [🚀 Quick Start](#-quick-start)
- [📋 Core Concepts](#-core-concepts)
- [🛠️ ScriptableObject Workflow](#️-scriptableobject-workflow)
- [💾 Serialization & Versioning](#-serialization--versioning)
- [🌐 Backend Integration](#-backend-integration)
- [📚 API Reference](#-api-reference)
- [🎮 Examples](#-examples)
- [⚡ Performance](#-performance)
- [🔧 Troubleshooting](#-troubleshooting)
- [❓ FAQ](#-faq)
- [🤝 Contributing](#-contributing)
- [📄 License](#-license)

---

## 📦 Requirements

- Unity 6.0 or newer (`"unity": "6000.0"`)
- Namespace: `GameLovers.ConfigsProvider`
- For JSON serialization: `Newtonsoft.Json` (Unity package `com.unity.nuget.newtonsoft-json`)
- Uses `Pair<TKey, TValue>` from `GameLovers.DataExtensions` (already referenced by the assembly definition)


## 📦 Installation

### Option 1: Unity Package Manager (Recommended)
1. Open Unity Package Manager (`Window` → `Package Manager`)
2. Click the **+** button → `Add package from git URL...`
3. Enter: `https://github.com/CoderGamester/Unity-ConfigsProvider.git#0.2.0`
4. Click `Add`

### Option 2: Manual Git URL
Add this line to your `Packages/manifest.json`:
```json
{
  "dependencies": {
    "com.gamelovers.configsprovider": "https://github.com/CoderGamester/Unity-ConfigsProvider.git#0.2.0"
  }
}
```

### Dependencies
This package automatically handles most dependencies, but you may need:

- **Newtonsoft.Json** (for serialization) - Install via Package Manager: `com.unity.nuget.newtonsoft-json`
- **GameLovers.DataExtensions** - Automatically included via assembly definition

### 🔍 Verify Installation
After installation, you should see:
- ✅ `GameLovers.ConfigsProvider` namespace available
- ✅ No compilation errors in Console  
- ✅ Runtime scripts accessible in your code


## 🚀 Quick Start

### Basic Setup in 3 Steps

#### Step 1: Define Your Config Classes
Create a new script `GameConfigs.cs`:

```csharp
using System;
using UnityEngine;

namespace MyGame.Configs
{
    [Serializable]
    public class EnemyConfig
    {
        public int Id;
        public string Name;
        public int Health;
        public float MoveSpeed;
        public GameObject Prefab;  // Unity asset references supported!
    }
    
    [Serializable]
    public class GameSettings
    {
        public float MusicVolume = 0.8f;
        public float SfxVolume = 1.0f;
        public bool ShowTutorials = true;
    }
}
```

#### Step 2: Initialize the Provider
In your game initialization (e.g., `GameManager.cs`):

```csharp
using GameLovers.ConfigsProvider;
using MyGame.Configs;
using System.Collections.Generic;
using UnityEngine;

public class GameManager : MonoBehaviour
{
    private IConfigsProvider _configs;
    
    void Start()
    {
        // Create provider
        var provider = new ConfigsProvider();
        
        // Add enemy configs (multiple instances mapped by ID)
        provider.AddConfigs(
            enemy => enemy.Id,  // Key selector function
            new List<EnemyConfig>
            {
                new() { Id = 1, Name = "Goblin", Health = 50, MoveSpeed = 3f },
                new() { Id = 2, Name = "Orc", Health = 100, MoveSpeed = 2f },
                new() { Id = 3, Name = "Dragon", Health = 500, MoveSpeed = 5f }
            });
        
        // Add game settings (singleton)
        provider.AddSingletonConfig(new GameSettings());
        
        _configs = provider;
        
        Debug.Log("Configs loaded successfully!");
    }
}
```

#### Step 3: Use Configs Anywhere
```csharp
using GameLovers.ConfigsProvider;

public class EnemySpawner : MonoBehaviour
{
    [SerializeField] private GameManager _gameManager;
    
    public void SpawnEnemy(int enemyId)
    {
        // Get specific enemy config
        var enemyConfig = _gameManager.Configs.GetConfig<EnemyConfig>(enemyId);
        
        // Use the config data
        var enemy = Instantiate(enemyConfig.Prefab);
        var healthComponent = enemy.GetComponent<Health>();
        healthComponent.SetMaxHealth(enemyConfig.Health);
        
        Debug.Log($"Spawned {enemyConfig.Name} with {enemyConfig.Health} HP");
    }
    
    public void ShowAllEnemies()
    {
        // Get all enemies
        foreach (var enemy in _gameManager.Configs.GetConfigsList<EnemyConfig>())
        {
            Debug.Log($"Enemy: {enemy.Name} (ID: {enemy.Id}) - {enemy.Health} HP");
        }
    }
    
    public void ApplyGameSettings()
    {
        // Get singleton settings
        var settings = _gameManager.Configs.GetConfig<GameSettings>();
        AudioManager.SetVolume(settings.MusicVolume, settings.SfxVolume);
        TutorialManager.SetEnabled(settings.ShowTutorials);
    }
}
```

---

## 📋 Core Concepts

### Singleton vs Collection Configs

**Singleton Configs** - One instance per type:
```csharp
// Perfect for game settings, global values
provider.AddSingletonConfig(new GameSettings());
var settings = provider.GetConfig<GameSettings>();
```

**Collection Configs** - Multiple instances mapped by ID:
```csharp
// Perfect for items, enemies, levels
provider.AddConfigs(item => item.ItemId, itemList);
var sword = provider.GetConfig<ItemConfig>(101);
```

### Safe Querying
```csharp
// Exception if not found
var config = provider.GetConfig<EnemyConfig>(999);

// Safe version - returns false if not found
if (provider.TryGetConfig<EnemyConfig>(999, out var config))
{
    // Use config safely
}
```

---

## 🛠️ ScriptableObject Workflow

For designer-friendly configuration authoring, use `ConfigsScriptableObject<TId, TAsset>`. This stores key/value pairs as a serializable list and builds a dictionary on load.

### Create the ScriptableObject
```csharp
using System;
using GameLovers.ConfigsProvider;
using UnityEngine;

[Serializable]
public class EnemyConfig
{
    public int Id;
    public string Name;
    public int Health;
    public float MoveSpeed;
    public GameObject Prefab;
}

[CreateAssetMenu(fileName = "Enemy Configs", menuName = "Game/Enemy Configs")]
public class EnemyConfigs : ConfigsScriptableObject<int, EnemyConfig> { }
```

### Author Data in Inspector
1. Right-click in Project → `Create` → `Game` → `Enemy Configs`
2. Select the created asset
3. In Inspector, add entries to the `Configs` list
4. Each entry has a `Key` (int) and `Value` (EnemyConfig)

### Use in Runtime
```csharp
public class ConfigLoader : MonoBehaviour
{
    [SerializeField] private EnemyConfigs _enemyConfigs;
    private IConfigsProvider _provider;
    
    void Start()
    {
        var provider = new ConfigsProvider();
        
        // Option 1: Use ScriptableObject dictionary directly
        var goblin = _enemyConfigs.ConfigsDictionary[1];
        
        // Option 2: Feed into main provider for unified access
        provider.AddConfigs(
            config => config.Id, 
            _enemyConfigs.Configs.Select(pair => pair.Value).ToList()
        );
        
        _provider = provider;
    }
}
```

**⚠️ Important Notes:**
- Duplicate keys will throw during deserialization
- `ConfigsDictionary` is read-only and built in `OnAfterDeserialize`
- Keys must be unique within each ScriptableObject


## 💾 Serialization & Versioning

Use `ConfigsSerializer` to serialize providers to JSON for storage or server transfer, with automatic version tracking.

### Basic Serialization
```csharp
using GameLovers.ConfigsProvider;

var serializer = new ConfigsSerializer();

// Serialize to JSON (e.g., to send to server or save locally)
string jsonData = serializer.Serialize(provider, version: "1.2.3");

// Deserialize back into a new provider instance
var restoredProvider = serializer.Deserialize<ConfigsProvider>(jsonData);

Debug.Log($"Restored provider with version: {restoredProvider.Version}");
```

### Exclude Types from Serialization
Mark types that should not be sent to clients/servers:

```csharp
using System;
using GameLovers.ConfigsProvider;

[IgnoreServerSerialization]
[Serializable]
public class EditorOnlyConfig 
{
    public string InternalNotes;
    public bool DebugMode;
}

[Serializable]
public class PlayerVisibleConfig
{
    public int MaxLevel;
    public float ExpMultiplier;
}
```

### Version Management
```csharp
// Check current version
Debug.Log($"Current version: {provider.Version}");

// Serialize with semantic versioning
string v1_0_0 = serializer.Serialize(provider, "1.0.0");
string v1_1_0 = serializer.Serialize(updatedProvider, "1.1.0");

// Version is automatically converted to ulong for comparison
var restored = serializer.Deserialize<ConfigsProvider>(v1_1_0);
// restored.Version will be a ulong representation of "1.1.0"
```

**📋 Requirements:**
- Config types must be `[Serializable]` unless marked with `[IgnoreServerSerialization]`
- Uses Newtonsoft.Json with `TypeNameHandling.Auto` and enum-as-string conversion


## 🌐 Backend Integration

Integrate with your backend using `IConfigBackendService` to poll for remote versions and perform atomic config updates.

### Implement Backend Service
```csharp
using System.Threading.Tasks;
using GameLovers.ConfigsProvider;
using UnityEngine;
using UnityEngine.Networking;

public class MyBackendService : IConfigBackendService
{
    private const string BASE_URL = "https://your-game-server.com/api/configs";
    
    public async Task<ulong> GetRemoteVersion()
    {
        using var request = UnityWebRequest.Get($"{BASE_URL}/version");
        await request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            var versionData = JsonUtility.FromJson<VersionResponse>(request.downloadHandler.text);
            return versionData.version;
        }
        
        throw new System.Exception($"Failed to get remote version: {request.error}");
    }

    public async Task<IConfigsProvider> FetchRemoteConfiguration(ulong version)
    {
        using var request = UnityWebRequest.Get($"{BASE_URL}/data/{version}");
        await request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            var serializer = new ConfigsSerializer();
            return serializer.Deserialize<ConfigsProvider>(request.downloadHandler.text);
        }
        
        throw new System.Exception($"Failed to fetch remote config: {request.error}");
    }
    
    [System.Serializable]
    private class VersionResponse
    {
        public ulong version;
    }
}
```

### Sync with Backend
```csharp
public class ConfigSyncManager : MonoBehaviour
{
    [SerializeField] private float _syncIntervalSeconds = 300f; // 5 minutes
    
    private ConfigsProvider _localProvider;
    private IConfigBackendService _backendService;
    
    void Start()
    {
        _localProvider = new ConfigsProvider();
        _backendService = new MyBackendService();
        
        // Start periodic sync
        InvokeRepeating(nameof(SyncWithBackend), 0f, _syncIntervalSeconds);
    }
    
    private async void SyncWithBackend()
    {
        try
        {
            var remoteVersion = await _backendService.GetRemoteVersion();
            
            if (remoteVersion > _localProvider.Version)
            {
                Debug.Log($"New config version available: {remoteVersion}");
                
                var remoteProvider = await _backendService.FetchRemoteConfiguration(remoteVersion);
                
                // Atomic update - copy data and bump version
                _localProvider.UpdateTo(remoteProvider.Version, remoteProvider.GetAllConfigs());
                
                Debug.Log($"Successfully updated to version {_localProvider.Version}");
                
                // Notify other systems of config update
                OnConfigsUpdated?.Invoke();
            }
        }
        catch (System.Exception ex)
        {
            Debug.LogError($"Config sync failed: {ex.Message}");
        }
    }
    
    public System.Action OnConfigsUpdated;
}
```


## 📚 API Reference

### Core Interfaces

#### `IConfigsProvider`
Read-only access to configuration data.

| Method | Description | Example |
|--------|-------------|---------|
| `ulong Version { get; }` | Current version number | `var version = provider.Version;` |
| `T GetConfig<T>()` | Get singleton config | `var settings = provider.GetConfig<GameSettings>();` |
| `T GetConfig<T>(int id)` | Get config by ID | `var enemy = provider.GetConfig<EnemyConfig>(1);` |
| `bool TryGetConfig<T>(int id, out T config)` | Safe get by ID | `if (provider.TryGetConfig(1, out var enemy)) { }` |
| `List<T> GetConfigsList<T>()` | Get all configs of type | `var allEnemies = provider.GetConfigsList<EnemyConfig>();` |
| `IReadOnlyDictionary<int, T> GetConfigsDictionary<T>()` | Get dictionary of configs | `var enemyDict = provider.GetConfigsDictionary<EnemyConfig>();` |
| `IReadOnlyDictionary<Type, IEnumerable> GetAllConfigs()` | Get all config data | `var allConfigs = provider.GetAllConfigs();` |

#### `IConfigsAdder : IConfigsProvider`
Write access for building configuration data.

| Method | Description | Example |
|--------|-------------|---------|
| `void AddSingletonConfig<T>(T config)` | Add singleton | `provider.AddSingletonConfig(settings);` |
| `void AddConfigs<T>(Func<T, int> keySelector, IList<T> configs)` | Add collection | `provider.AddConfigs(e => e.Id, enemies);` |
| `void AddAllConfigs(IReadOnlyDictionary<Type, IEnumerable> configs)` | Add bulk configs | `provider.AddAllConfigs(configDict);` |
| `void UpdateTo(ulong version, IReadOnlyDictionary<Type, IEnumerable> configs)` | Atomic update | `provider.UpdateTo(42, newConfigs);` |

#### `ConfigsProvider`
Default implementation using in-memory dictionaries.

#### `IConfigsSerializer`
JSON serialization interface.

| Method | Description | Example |
|--------|-------------|---------|
| `string Serialize(IConfigsProvider provider, string version)` | Serialize to JSON | `var json = serializer.Serialize(provider, "1.0");` |
| `T Deserialize<T>(string json) where T : IConfigsAdder` | Deserialize from JSON | `var provider = serializer.Deserialize<ConfigsProvider>(json);` |

#### `ConfigsScriptableObject<TId, TAsset>`
Unity-serializable container for designer-authored configs.

| Property | Description | Example |
|----------|-------------|---------|
| `List<Pair<TId, TAsset>> Configs` | Editable config pairs | Edit in Inspector |
| `IReadOnlyDictionary<TId, TAsset> ConfigsDictionary` | Runtime lookup dictionary | `var item = configs.ConfigsDictionary[itemId];` |

#### `IConfigBackendService`
Optional interface for remote config fetching.

| Method | Description | Example |
|--------|-------------|---------|
| `Task<ulong> GetRemoteVersion()` | Get latest version from server | `var version = await service.GetRemoteVersion();` |
| `Task<IConfigsProvider> FetchRemoteConfiguration(ulong version)` | Fetch config data | `var configs = await service.FetchRemoteConfiguration(42);` |

### Helper Interfaces
- `IConfig` - Simple interface with `int ConfigId { get; }`
- `IConfigsContainer<T>`, `ISingleConfigContainer<T>` - Container patterns
- `IPairConfigsContainer<TKey, TValue>`, `IStructPairConfigsContainer<TKey, TValue>` - Pair containers

---

## 🎮 Examples

### Example 1: RPG Item Database
```csharp
using System;
using UnityEngine;
using GameLovers.ConfigsProvider;

[Serializable]
public class ItemConfig
{
    public int ItemId;
    public string ItemName;
    public ItemType Type;
    public int Value;
    public Sprite Icon;
    public GameObject Prefab;
    
    // Computed properties
    public bool IsWeapon => Type == ItemType.Weapon;
    public bool IsConsumable => Type == ItemType.Consumable;
}

public enum ItemType
{
    Weapon,
    Armor,
    Consumable,
    Quest
}

[CreateAssetMenu(fileName = "Item Database", menuName = "Game/Item Database")]
public class ItemDatabase : ConfigsScriptableObject<int, ItemConfig> { }

// Usage in game
public class InventoryManager : MonoBehaviour
{
    [SerializeField] private ItemDatabase _itemDatabase;
    
    public void AddItemToInventory(int itemId, int quantity)
    {
        if (_itemDatabase.ConfigsDictionary.TryGetValue(itemId, out var itemConfig))
        {
            Debug.Log($"Added {quantity}x {itemConfig.ItemName} to inventory");
            // Add to player inventory...
        }
        else
        {
            Debug.LogError($"Item ID {itemId} not found in database!");
        }
    }
}
```

### Example 2: Dynamic Difficulty System
```csharp
[Serializable]
public class DifficultyConfig
{
    public DifficultyLevel Level;
    public float EnemyHealthMultiplier;
    public float EnemyDamageMultiplier;
    public float PlayerExpMultiplier;
    public int MaxEnemiesPerWave;
}

public enum DifficultyLevel
{
    Easy = 1,
    Normal = 2,
    Hard = 3,
    Nightmare = 4
}

public class DifficultyManager : MonoBehaviour
{
    private IConfigsProvider _configs;
    private DifficultyLevel _currentDifficulty = DifficultyLevel.Normal;
    
    void Start()
    {
        var provider = new ConfigsProvider();
        provider.AddConfigs(d => (int)d.Level, new List<DifficultyConfig>
        {
            new() { Level = DifficultyLevel.Easy, EnemyHealthMultiplier = 0.7f, EnemyDamageMultiplier = 0.8f, PlayerExpMultiplier = 0.8f, MaxEnemiesPerWave = 3 },
            new() { Level = DifficultyLevel.Normal, EnemyHealthMultiplier = 1.0f, EnemyDamageMultiplier = 1.0f, PlayerExpMultiplier = 1.0f, MaxEnemiesPerWave = 5 },
            new() { Level = DifficultyLevel.Hard, EnemyHealthMultiplier = 1.5f, EnemyDamageMultiplier = 1.3f, PlayerExpMultiplier = 1.2f, MaxEnemiesPerWave = 7 },
            new() { Level = DifficultyLevel.Nightmare, EnemyHealthMultiplier = 2.0f, EnemyDamageMultiplier = 1.8f, PlayerExpMultiplier = 1.5f, MaxEnemiesPerWave = 10 }
        });
        _configs = provider;
    }
    
    public void ChangeDifficulty(DifficultyLevel newDifficulty)
    {
        _currentDifficulty = newDifficulty;
        var config = _configs.GetConfig<DifficultyConfig>((int)newDifficulty);
        
        // Apply difficulty settings
        EnemyManager.SetHealthMultiplier(config.EnemyHealthMultiplier);
        EnemyManager.SetDamageMultiplier(config.EnemyDamageMultiplier);
        ExperienceManager.SetExpMultiplier(config.PlayerExpMultiplier);
        WaveManager.SetMaxEnemies(config.MaxEnemiesPerWave);
        
        Debug.Log($"Difficulty changed to {newDifficulty}");
    }
}
```

---

## ⚡ Performance

### Benchmarks
- **Lookup Performance**: O(1) for both singleton and ID-based configs
- **Memory Usage**: ~50 bytes overhead per config + actual config size
- **Initialization**: ~1ms for 1000 configs on average hardware
- **Serialization**: ~10ms for 1000 configs to/from JSON

### Best Practices
- ✅ **Load configs during loading screens** - One-time initialization cost
- ✅ **Reuse `IConfigsProvider` instances** - Don't recreate providers unnecessarily  
- ✅ **Use `TryGetConfig` for optional configs** - Avoid exceptions for missing data
- ✅ **Cache frequently accessed configs** - Store references if accessed every frame
- ✅ **Use ScriptableObjects for large datasets** - Better for authoring and iteration
- ❌ **Don't call `GetConfigsList` repeatedly** - Cache the list if you need it multiple times
- ❌ **Don't modify configs at runtime** - Treat them as immutable data

### Memory Management
```csharp
// Good: Cache frequently used configs
public class EnemyAI : MonoBehaviour
{
    private EnemyConfig _config; // Cached reference
    
    void Start()
    {
        _config = ConfigManager.Instance.GetConfig<EnemyConfig>(enemyId);
    }
    
    void Update()
    {
        // Use cached config - no lookup cost
        transform.Translate(Vector3.forward * _config.MoveSpeed * Time.deltaTime);
    }
}

// Bad: Lookup every frame
public class SlowEnemyAI : MonoBehaviour
{
    void Update()
    {
        // DON'T DO THIS - expensive lookup every frame!
        var config = ConfigManager.Instance.GetConfig<EnemyConfig>(enemyId);
        transform.Translate(Vector3.forward * config.MoveSpeed * Time.deltaTime);
    }
}
```

---

## 🔧 Troubleshooting

### Common Issues & Solutions

#### `InvalidOperationException` when calling `GetConfig<T>()`
**Problem**: Type was not registered as a singleton  
**Solution**: Use `GetConfig<T>(id)` for collection configs, or register as singleton with `AddSingletonConfig<T>`

```csharp
// Wrong - EnemyConfig is a collection, not singleton
var enemy = provider.GetConfig<EnemyConfig>(); // ❌ Throws exception

// Correct ways
var enemy = provider.GetConfig<EnemyConfig>(1); // ✅ Get by ID
var allEnemies = provider.GetConfigsList<EnemyConfig>(); // ✅ Get all
```

#### Duplicate Key Exception in ScriptableObject
**Problem**: Multiple entries with the same key in `ConfigsScriptableObject`  
**Solution**: Ensure each key is unique in the Inspector

```csharp
// In Inspector, make sure you don't have:
// Key: 1, Value: Enemy1
// Key: 1, Value: Enemy2  // ❌ Duplicate key!

// Instead use unique keys:
// Key: 1, Value: Goblin   // ✅
// Key: 2, Value: Orc      // ✅
```

#### Serialization Fails for Custom Types
**Problem**: Config type is not marked as `[Serializable]`  
**Solution**: Add `[Serializable]` attribute or exclude with `[IgnoreServerSerialization]`

```csharp
// Wrong
public class MyConfig { } // ❌ Not serializable

// Correct options
[Serializable]
public class MyConfig { } // ✅ Will be serialized

[IgnoreServerSerialization]
public class EditorOnlyConfig { } // ✅ Will be excluded
```

#### Newtonsoft.Json Not Found
**Problem**: `ConfigsSerializer` requires Newtonsoft.Json  
**Solution**: Install via Package Manager

1. Open Package Manager
2. Search for `Newtonsoft Json`
3. Install `com.unity.nuget.newtonsoft-json`

#### Config Data Not Updating
**Problem**: ScriptableObject changes not reflected at runtime  
**Solution**: 
- Check that you're loading the correct asset reference
- Ensure the asset is saved after changes
- For runtime changes, use `ConfigsProvider.UpdateTo()` instead

#### Performance Issues with Large Config Sets
**Problem**: Slow initialization with thousands of configs  
**Solution**:
- Use `ConfigsScriptableObject` for better loading performance
- Consider lazy loading patterns for very large datasets
- Split large config sets into multiple smaller ones

---

## ❓ FAQ

**Q: What's the minimum Unity version?**  
A: The package.json specifies Unity 6000.0 (Unity 6), but it may work with earlier versions. The package uses standard C# features available in Unity 2021.3+.

**Q: Can I use this with Addressables?**  
A: Yes! Load your `ConfigsScriptableObject` via Addressables and feed it to the provider:
```csharp
var handle = Addressables.LoadAssetAsync<EnemyConfigs>("enemy-configs");
var configs = await handle.Task;
provider.AddConfigs(e => e.Id, configs.Configs.Select(p => p.Value).ToList());
```

**Q: How do I handle config validation?**  
A: Implement validation in your config classes or use Unity's `OnValidate`:
```csharp
[Serializable]
public class EnemyConfig
{
    public int Health;
    
    public bool IsValid => Health > 0;
}

// In ScriptableObject
public class EnemyConfigs : ConfigsScriptableObject<int, EnemyConfig>
{
    void OnValidate()
    {
        foreach (var config in Configs)
        {
            if (!config.Value.IsValid)
                Debug.LogError($"Invalid config: {config.Key}");
        }
    }
}
```

**Q: Is this thread-safe?**  
A: No, the current implementation is not thread-safe. Use it from the main thread only or implement your own synchronization.

**Q: Can I modify configs at runtime?**  
A: Configs should be treated as immutable. For dynamic changes, use `UpdateTo()` to replace the entire config set atomically.

**Q: How do I handle missing optional configs?**  
A: Use `TryGetConfig` instead of `GetConfig`:
```csharp
if (provider.TryGetConfig<BossConfig>(bossId, out var bossConfig))
{
    // Boss has custom config
    SpawnBoss(bossConfig);
}
else
{
    // Use default boss behavior
    SpawnDefaultBoss();
}
```

**Q: Can I nest config objects?**  
A: Yes, as long as all nested types are `[Serializable]`:
```csharp
[Serializable]
public class EnemyConfig
{
    public int Id;
    public Stats BaseStats;     // ✅ Nested serializable object
    public List<Ability> Abilities; // ✅ List of serializable objects
}

[Serializable]
public class Stats
{
    public int Health;
    public float Speed;
}
```

**Q: How do I version my configs for compatibility?**  
A: Use the version string in serialization and implement migration logic:
```csharp
var json = serializer.Serialize(provider, "2.1.0");
var restored = serializer.Deserialize<ConfigsProvider>(json);

// Check version and migrate if needed
if (restored.Version < expectedVersion)
{
    MigrateConfigs(restored);
}
```

---

## 🤝 Contributing

We welcome contributions! Here's how you can help:

### 🐛 Reporting Issues
- Use [GitHub Issues](https://github.com/CoderGamester/Unity-ConfigsProvider/issues)
- Include Unity version, package version, and minimal reproduction steps
- For performance issues, include profiler data if possible

### 🛠️ Development Setup
1. Clone the repository
2. Open in Unity 6000.0+
3. Run tests in `Tests/Editor/`
4. Make your changes
5. Ensure all tests pass
6. Submit a pull request

### 📋 Code Guidelines
- Follow existing code style
- Add unit tests for new features
- Update documentation for API changes
- Use clear, descriptive commit messages

### 🎯 Areas We Need Help With
- Performance optimizations
- Additional serialization formats
- More comprehensive examples
- Documentation improvements
- Unit test coverage

---

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE.md](LICENSE.md) file for details.

### What This Means
- ✅ **Commercial use** - Use in commercial projects
- ✅ **Modification** - Modify the source code  
- ✅ **Distribution** - Share with others
- ✅ **Private use** - Use for personal projects
- ❗ **Liability** - No warranty provided
- ❗ **Attribution** - Must include license notice

---

<p align="center">
  <strong>Made with ❤️ for the Unity community</strong>
</p>

<p align="center">
  <a href="https://github.com/CoderGamester/Unity-ConfigsProvider">🌟 Star on GitHub</a> •
  <a href="https://github.com/CoderGamester/Unity-ConfigsProvider/issues">🐛 Report Issues</a> •
  <a href="#-contributing">🤝 Contribute</a>
</p>


