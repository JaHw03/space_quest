# SPACE QUEST - GAME DESIGN DOCUMENT (GDD)

## 📋 MỤC LỤC
1. [OVERVIEW](#overview)
2. [GAME ELEMENTS](#game-elements)
3. [GAME MECHANICS](#game-mechanics)
4. [PROGRESSION SYSTEM](#progression-system)
5. [ENEMIES & BOSSES](#enemies--bosses)
6. [OBSTACLES](#obstacles)
7. [WEAPONS & UPGRADES](#weapons--upgrades)
8. [UI & HUD](#ui--hud)
9. [AUDIO](#audio)
10. [INPUT SYSTEM](#input-system)
11. [CAMERA & EFFECTS](#camera--effects)
12. [LEVEL DESIGN](#level-design)
13. [WIN/LOSE CONDITIONS](#winlose-conditions)
14. [CORE LOOP](#core-loop)
15. [TECHNICAL ARCHITECTURE](#technical-architecture)

---

## OVERVIEW

### Thông Tin Cơ Bản
| Thuộc Tính | Giá Trị |
|-----------|---------|
| **Tên Game** | Space Quest |
| **Thể Loại** | Action / Horizontal Shoot-em-up |
| **Nền Tảng** | Unity 2D |
| **Thị Trường** | PC / Web |
| **Số Người Chơi** | Single Player |
| **Lối Chơi** | Survival/Endless Mode |

### Mô Tả Game
Space Quest là một game bắn súng chiều ngang 2D trong đó người chơi điều khiển một tàu vũ trụ bay trong không gian vô tận. Mục đích chính là:
- **Sống sót càng lâu càng tốt** trước các dòng kẻ thù liên tục
- **Tiêu diệt kẻ thù** để nhận kinh nghiệm (Experience)
- **Nâng cấp** vũ khí, sức khỏe thông qua hệ thống leveling
- **Tránh chướng ngại vật** (tiểu hành tinh) hoặc bắn chúng

### Lõi Trò Chơi (Core Gameplay Loop)
```
Di chuyển → Bắn súng → Né tránh → Tiêu diệt → Kinh nghiệm → Nâng cấp → Lặp lại
```

### Giá Trị Thương Hiệu
- **Challenge**: Độ khó tăng dần, enemies chính xác
- **Replayability**: Endless mode, mỗi lần chơi khác nhau
- **Instant Gratification**: Level up liên tục, feedback hiệu ứng ngay lập tức

---

## GAME ELEMENTS

### 1. PLAYER (Người Chơi)

#### Thông Tin Cơ Bản
| Thuộc Tính | Giá Trị | Chi Tiết |
|-----------|---------|---------|
| **Type** | Main Controllable Character | Tàu vũ trụ |
| **Appearance** | Sprite-based 2D | Có animation di chuyển 4 hướng |
| **Size** | ~0.8-1.0 unit | Kích thước tiêu chuẩn |
| **Collision** | CircleCollider2D | Radius ~0.5 |
| **Physics** | Rigidbody2D (Dynamic) | Gravity Scale: 0 |

#### Thuộc Tính & Stats
| Stat | Giá Trị Ban Đầu | Max | Tăng Mỗi Level | Ghi Chú |
|------|----------------|-----|----------------|--------|
| **Health** | 5 | Không giới hạn | +1 / level | Game Over khi HP = 0 |
| **Move Speed** | 2.0 units/s | 2.0 | Không tăng | Speed cố định |
| **Max Energy** | 50 | 50 | Không tăng | Dùng cho Boost |
| **Energy Regen** | 0.05/frame | - | Không tăng | Tự phục hồi |
| **Current Level** | 1 | 30 | - | Max 30 levels |
| **Experience** | 0 | Không giới hạn | - | Nhận từ tiêu diệt enemies |

#### Chuyển Động & Di Chuyển
- **Input**: WASD + Arrow Keys (4 hướng)
- **Velocity**: `linearVelocity = playerDirection × moveSpeed`
- **Animation**: Blend 4 hướng (moveX, moveY parameters)
- **Boundary**: Có thể di chuyển tự do, không có ranh giới bản đồ

#### Tương Tác Vật Lý
| Đối Tượng | Hành Động | Kết Quả |
|-----------|----------|--------|
| **Asteroid** | Va chạm | -1 Health |
| **Enemy** | Va chạm | -1 Health |
| **Phaser Bullet** | Bắn | Không ảnh hưởng |

#### Sprite & Animation
- **Idle**: Sprite tĩnh
- **Move Up**: Animation lên (moveY = 1)
- **Move Down**: Animation xuống (moveY = -1)
- **Move Left**: Animation trái (moveX = -1) + Player_Left sprite
- **Move Right**: Animation phải (moveX = 1)
- **Boost**: Engine effect particle chạy + animator.boosting = true

### 2. ENERGY SYSTEM (Hệ Thống Năng Lượng)

#### Mục Đích
Giới hạn khả năng sử dụng Boost của người chơi, tạo ra chiến lược

#### Cơ Chế
| Thành Phần | Giá Trị |
|-----------|---------|
| **Max Energy** | 50 |
| **Regen Rate** | +0.05/frame (khi không boost) |
| **Boost Cost** | -0.5/frame (khi boost) |
| **Min để Boost** | 10 (Yêu cầu tối thiểu) |
| **Display** | UI Slider + Text (hiện tại/max) |

#### States
1. **Resting**: Energy từ từ tăng (+0.05/frame)
2. **Boosting**: Energy giảm nhanh (-0.5/frame)
3. **Depleted**: Không đủ energy, forced exit boost

### 3. HEALTH SYSTEM (Hệ Thống Máu)

#### Cơ Chế
- **Max Health**: 5 (ban đầu) → tăng +1 mỗi level up
- **Damage**: -1 HP mỗi lần va chạm
- **Healing**: Không có healing, chỉ có level up restore
- **Display**: UI Slider + Text (hiện tại/max)
- **Game Over**: Kích hoạt khi Health ≤ 0

#### Hiệu Ứng Khi Bị Đánh
1. Health giảm
2. **Flash White**: Sprite đổi sang material trắng (0.2s)
3. **Sound**: Âm thanh hit đặc biệt
4. **Không stun**: Người chơi vẫn có thể di chuyển

### 4. EXPERIENCE & LEVEL SYSTEM

#### Cấu Trúc Level
| Level | EXP Cần | Mô Tả |
|-------|---------|-------|
| 1 | 0 | Khởi đầu |
| 2 | 15 | |
| 3 | 31 | Formula: `playerLevels[i-1] × 1.1 + 15` |
| ... | ... | Tăng dần theo công thức |
| 30 | ~15000+ | Max Level |

#### Khi Level Up
```
1. Experience giảm (trừ đi yêu cầu của level hiện tại)
2. Max Health tăng +1
3. Health reset = Max Health
4. Phaser Weapon Level Up
5. UI cập nhật
```

#### Kinh Nghiệm Từ Enemies
| Enemy | EXP |
|-------|-----|
| Asteroid | 1 |
| Locustmorph | TBD (base: 5) |
| Boss 1 | TBD (base: 50) |
| Boss 2 | TBD (base: 50) |

### 5. BOOST MECHANIC (Cơ Chế Tăng Tốc)

#### Chức Năng
Tăng tốc độ toàn bộ game, cho phép người chơi:
- Di chuyển nhanh hơn
- Né tránh kẻ thù dễ hơn
- Nhưng Phaser Weapon kém hiệu quả (enemies cũng nhanh hơn)

#### Stats
| Thuộc Tính | Giá Trị |
|-----------|---------|
| **World Speed Multiplier** | 7x (so với 1x bình thường) |
| **Energy Cost** | -0.5/frame |
| **Min Energy to Activate** | 10 |
| **Particle Effect** | Engine effect chạy |
| **Sound** | "fire" audio |
| **Animation** | animator.boosting = true |

#### Input & Control
- **Activate**: Space hoặc GamePad Fire2 (nhấn xuống)
- **Deactivate**: Space hoặc GamePad Fire2 (thả ra)
- **Auto Exit**: Khi Energy < 0.5f

#### Impact trên Gameplay
```csharp
// Trong FixedUpdate:
if (boosting) {
    if (energy >= 0.5f) energy -= 0.5f;
    else ExitBoost();
}
```

---

## GAME MECHANICS

### 1. OBJECT POOLING

#### Mục Đích
Tối ưu hiệu năng bằng cách reuse objects thay vì Instantiate/Destroy liên tục

#### Cách Hoạt Động
```csharp
// Khởi tạo pool
ObjectPooler pooler = new ObjectPooler();
pooler.prefab = bulletPrefab;
pooler.poolSize = 100;
pooler.CreatePool();

// Sử dụng
GameObject bullet = pooler.GetPooledObject();
bullet.SetActive(true);

// Sau khi dùng xong
bullet.SetActive(false); // Quay lại pool
```

#### Pools Trong Game
| Pool Name | Prefab | Size | Loại |
|-----------|--------|------|------|
| BulletPool | Phaser Bullet | 200+ | Bullet |
| Boom1Pool | Explosion (Small) | 50 | Destruction |
| Boom2Pool | Explosion (Medium) | 50 | Destruction |
| Boom3Pool | Explosion (Large) | 30 | Destruction |
| LocustPopPool | Locust Destroy | 50 | Enemy FX |
| Boss1Pool | Boss 1 Enemy | 1 | Boss |
| Boss2Pool | Boss 2 Enemy | 1 | Boss |

### 2. WAVE-BASED SPAWNING

#### Cấu Trúc Wave
```csharp
public class Wave {
    public ObjectPooler pool;           // Pool để spawn từ
    public float spawnInterval;         // Khoảng cách giữa spawn (giây)
    public int objectsPerWave;          // Số lượng spawn mỗi wave
    public int spawnedObjectsCount;     // Đã spawn bao nhiêu
}
```

#### Các Wave Hiện Có
| Wave # | Enemy Type | Spawn Interval | Count | Difficulty |
|--------|-----------|----------------|-------|------------|
| 0 | Asteroid | 1.0s | 10 | Easy |
| 1 | Locustmorph | 1.5s | 8 | Easy-Med |
| 2 | Asteroid + Locust | 0.8s | 12 | Medium |
| ... | ... | ... | ... | ... |
| N | Repeat từ Wave 0 | - | - | Endless |

#### Spawning Location
- **Position X**: minPos.x (cố định, từ phía bên phải màn hình)
- **Position Y**: Random từ minPos.y đến maxPos.y (-5 đến 5)

#### Wave Progression
```
1. Tính thời gian spawn: spawnTimer -= GameManager.adjustedWorldSpeed
2. Nếu spawnTimer <= 0:
   - Lấy object từ pool
   - Set position
   - SetActive(true)
   - spawnedObjectsCount++
3. Nếu spawnedObjectsCount >= objectsPerWave:
   - Reset counter
   - Chuyển sang wave tiếp theo
   - Nếu hết wave, quay lại wave 0
```

### 3. COLLISION & DAMAGE SYSTEM

#### Collision Types
| Type | Trigger | Kết Quả |
|------|---------|--------|
| Player + Asteroid | Collision2D | Player -1 HP |
| Player + Enemy | Collision2D | Player -1 HP |
| Bullet + Enemy | Collision2D | Enemy -damage HP |
| Bullet + Asteroid | Collision2D | Asteroid -1 HP |
| Boss + Asteroid | Collision2D | Asteroid -damage HP |

#### Damage Calculation
```csharp
// Enemy to Player
PlayerController.TakeDamage(damage: int)
    health -= damage
    flashWhite.Flash(0.2s)
    if (health <= 0) GameOver()

// Bullet to Enemy/Obstacle
Enemy.TakeDamage(damage: int)
    lives -= damage
    if (lives <= 0) {
        CreateExplosion()
        GiveExperience()
        SetActive(false) // Return to pool
    }
```

### 4. FLASH WHITE EFFECT

#### Mục Đích
Visual feedback khi nhân vật/enemy bị đánh

#### Cơ Chế
```csharp
public void Flash() {
    spriteRenderer.material = whiteMaterial; // Load "Materials/mWhite"
    StartCoroutine(ResetMaterial(0.2f));    // Sau 0.2s đổi lại bình thường
}

IEnumerator ResetMaterial() {
    yield return new WaitForSeconds(0.2f);
    spriteRenderer.material = defaultMaterial;
}
```

#### Timing
| Khi Nào | Duration |
|--------|----------|
| Player bị đánh | 0.2s |
| Enemy bị đánh | 0.2s |
| Asteroid bị đánh | 0.2s |

---

## PROGRESSION SYSTEM

### 1. LEVEL UP FLOW

```
Collect EXP → Reach EXP Threshold → LevelUp() triggered
                                           ↓
                         Max Health +1
                         Health = Max Health
                         Phaser Weapon Level Up
                         EXP Reset
                         UI Update
```

### 2. WEAPON PROGRESSION

#### Phaser Weapon Levels
| Level | Speed | Damage | Size | Amount | Range | Notes |
|-------|-------|--------|------|--------|-------|-------|
| 1 | TBD | TBD | TBD | 1 bullet | 0 | Single shot |
| 2+ | Tăng | Tăng | Tăng | 2-3+ | Tăng | Spread pattern |
| ... | ... | ... | ... | ... | ... | ... |

#### Level Up Effect
```csharp
// Trong PhaserWeapon.LevelUp()
if (weaponLevel < stats.Count - 1) {
    weaponLevel++;
    // Stats tự động đọc từ stats[weaponLevel]
}
```

### 3. HEALTH PROGRESSION

| Level | Base HP | Total HP | Survival Impact |
|-------|---------|----------|-----------------|
| 1 | 5 | 5 | Dễ chết |
| 5 | 1 | 9 | Dung sai tăng |
| 10 | 1 | 14 | Chơi thoải mái hơn |
| 20 | 1 | 24 | Dễ xơi kẻ thù mạnh |
| 30 | 1 | 34 | Ông vua |

---

## ENEMIES & BOSSES

### 1. ENEMY (Base Class)

#### Thuộc Tính Chung
| Thuộc Tính | Kiểu | Mô Tả |
|-----------|-----|-------|
| **lives** | int | HP hiện tại |
| **maxLives** | int | HP tối đa |
| **damage** | int | Sát thương gây ra |
| **experienceToGive** | int | EXP drop |
| **speedX, speedY** | float | Vận tốc di chuyển |
| **hitSound** | AudioSource | Âm thanh bị đánh |
| **destroySound** | AudioSource | Âm thanh bị hủy |

#### Phương Thức Ảo
```csharp
public virtual void Awake() { }
public virtual void OnEnable() { }
public virtual void Start() { }
public virtual void Update() { }
public virtual void OnCollisionEnter2D(Collision2D) { }
public virtual void TakeDamage(int damage) { }
```

### 2. BOSS 1 (Lữ đoàn chính)

#### Thông Tin
| Thuộc Tính | Giá Trị |
|-----------|---------|
| **Name** | Boss 1 / Chủ tàu |
| **Spawn Trigger** | Sau khi tiêu diệt 5 enemies thường |
| **Spawn Position** | (15, 0) - Phía bên phải |
| **Health** | TBD (cao, ví dụ 30-50 HP) |
| **Damage** | TBD (cao, ví dụ 5-10) |
| **Size** | ~2.0 units |
| **Experience** | TBD (ví dụ 50+) |

#### AI & States

**State 1: Charge**
- Tấn công nhanh về phía người chơi
- `speedX = -5f` (hướng trái)
- `speedY = 0`
- Duration: 0.6s - 1.3s ngẫu nhiên
- Animation: "charging" = true

**State 2: Patrol**
- Di chuyển chậm, tránh người chơi
- `speedX = 0`
- `speedY = Random(-1, 1)`
- Duration: 5s - 10s ngẫu nhiên
- Animation: "charging" = false

**State Switching**
```csharp
// Update hàng frame
if (switchTimer > 0) {
    switchTimer -= Time.deltaTime;
} else {
    // Sau khi timer hết, chuyển state
    if (charging && transform.position.x > playerPosition) {
        EnterPatrolState();
    } else {
        EnterChargeState();
    }
}

// Hoặc nếu player ở bên trái và boss ở bên phải
if (transform.position.x < playerPosition) {
    EnterChargeState();
}
```

#### Boundary Behavior
- **Top**: `if (y > 3)` → `speedY *= -1`
- **Bottom**: `if (y < -3)` → `speedY *= -1`
- **Left (Despawn)**: `if (x < -11)` → `SetActive(false)`

#### Special: Boost Reaction
```csharp
// Khi player boost, boss di chuyển chậm hơn
if (playerBoost && !charging) {
    moveX = GameManager.worldSpeed * -0.5f * Time.deltaTime;
} else {
    moveX = speedX * Time.deltaTime;
}
```

#### Sound & Effects
| Event | Sound |
|-------|-------|
| Spawn | bossSpawn |
| Enter Charge | bossCharge |
| Take Damage | hitArmor |
| Destroyed | boom2 (large explosion) |

### 3. BOSS 2 (Chủ tàu thứ 2)

#### Thông Tin
| Thuộc Tính | Giá Trị |
|-----------|---------|
| **Name** | Boss 2 |
| **Differences** | Speed, behavior khác |
| **Health** | TBD |
| **Size** | ~2.0 units |

#### AI & States

**State 1: Idle**
- Di chuyển chậm, thoải mái
- `speedX = 0.2f`
- `speedY = Random(-1.2, 1.2)`

**State 2: Charge**
- Tấn công nhanh
- `speedX = Random(3.5, 4)`
- `speedY = 0`

**Switching Logic**
```csharp
if (transform.position.x > 7.5f) {
    EnterIdleState();
}
else if (transform.position.x < -4 || 
         transform.position.x < playerPosition) {
    EnterChargeState();
}
```

### 4. LOCUSTMORPH (Kẻ thù bay nhỏ)

#### Thông Tin
| Thuộc Tính | Giá Trị |
|-----------|---------|
| **Name** | Locustmorph |
| **Size** | 0.7x0.7 unit |
| **Health** | TBD (thấp, ví dụ 2-3) |
| **Damage** | 1 |
| **Variants** | Multiple (2-3 biến thể sprite) |
| **Speed (Idle)** | 0.1-0.6 |
| **Speed (Charge)** | -4 đến -6 |

#### AI & States

**State 1: Idle** (Ban đầu)
- Di chuyển chậm, tự do
- `speedX = Random(0.1, 0.6)`
- `speedY = Random(-0.9, 0.9)`
- Sprite variant 0

**State 2: Charge** (HP < 50%)
- Di chuyển nhanh về trái (tấn công)
- `speedX = Random(-4, -6)`
- `speedY = 0`
- Sprite variant 1

**Triggering Charge**
```csharp
public override void TakeDamage(int damage) {
    base.TakeDamage(damage);
    if (lives <= maxLives * 0.5f) {
        EnterCharge();
    }
}
```

#### Variant System
```csharp
private List<Frames> frames; // Nhiều frame set

public class Frames {
    public Sprite[] sprites; // [0]: Idle, [1]: Charge
}

// Chọn variant ngẫu nhiên
enemyVariant = Random.Range(0, frames.Count);
spriteRenderer.sprite = frames[enemyVariant].sprites[stateIndex];
```

#### Boundary
- **Top/Bottom**: `if (y > 5 || y < -5)` → `speedY *= -1`

#### Sound
| Event | Sound |
|-------|-------|
| Take Damage | beetleHit |
| Destroyed | beetleDestroy |
| Enter Charge | locustCharge |

---

## OBSTACLES

### 1. ASTEROID (Tiểu hành tinh)

#### Thông Tin Cơ Bản
| Thuộc Tính | Giá Trị |
|-----------|---------|
| **Type** | Physics-based obstacle |
| **Size** | 0.6 - 1.0 (ngẫu nhiên scale) |
| **Health** | 3 HP |
| **Damage** | 1 HP (khi va player) |
| **Sprites** | 3 loại khác nhau |
| **Physics** | Rigidbody2D + velocity |
| **Collision** | BoxCollider2D |

#### Cơ Chế Tạo
```csharp
// OnEnable (mỗi lần activate từ pool)
lives = maxLives; // Reset HP = 3
transform.rotation = Quaternion.identity;
pushX = Random.Range(-1, 0);    // Tốc độ X ngẫu nhiên
pushY = Random.Range(-1, 1);    // Tốc độ Y ngẫu nhiên
rb.linearVelocity = new Vector2(pushX, pushY);

// Start
spriteRenderer.sprite = sprites[Random.Range(0, 3)]; // Random sprite
transform.localScale = new Vector2(randomScale, randomScale); // Random size
```

#### Damage Flow
```
Hit by Bullet
    ↓
TakeDamage(1, giveExperience: true)
    ↓
lives--
    ↓
if (lives > 0) {
    flashWhite.Flash(0.2s)
    hitRock sound
} else {
    destroyEffect (Boom2Pool)
    boom2 sound
    if (giveExperience) 
        PlayerController.GetExperience(1)
    SetActive(false) → Back to pool
}
```

#### Sound Effects
| Event | Sound |
|-------|-------|
| Take Damage | hitRock |
| Destroyed | boom2 |

#### Sprite Varieties
- Sprite 0: Type A asteroid
- Sprite 1: Type B asteroid
- Sprite 2: Type C asteroid

---

## WEAPONS & UPGRADES

### 1. PHASER WEAPON

#### Cấu Trúc
```csharp
public class Weapon : MonoBehaviour {
    public int weaponLevel;
    public List<WeaponStats> stats; // Cấp độ → stats
}

public class WeaponStats {
    public float speed;      // Tốc độ đạn
    public int damage;       // Sát thương
    public float size;       // Kích thước
    public float amount;     // Số lượng đạn (1, 2, 3...)
    public float range;      // Khoảng cách giữa các đạn
}
```

#### Firing Mechanics
```csharp
public void Shoot() {
    AudioManager.Instance.PlayModifiedSound(shoot);
    
    for (int i = 0; i < stats[weaponLevel].amount; i++) {
        GameObject bullet = bulletPool.GetPooledObject();
        float yPos = transform.position.y;
        
        // Nếu nhiều đạn, spread ra
        if (stats[weaponLevel].amount > 1) {
            float spacing = stats[weaponLevel].range / 
                          (stats[weaponLevel].amount - 1);
            yPos = transform.position.y - 
                   (stats[weaponLevel].range / 2) + 
                   i * spacing;
        }
        
        bullet.transform.position = new Vector2(transform.position.x, yPos);
        bullet.transform.localScale = new Vector2(size, size);
        bullet.SetActive(true);
    }
}
```

#### Leveling
```csharp
public void LevelUp() {
    if (weaponLevel < stats.Count - 1) {
        weaponLevel++;
    }
    // Stats tự động đọc từ stats[weaponLevel]
}
```

#### Bullet Behavior
- **Movement**: Tuyến tính về phía trước (speedX = stats.speed)
- **Damage**: Gây `stats.damage` sát thương cho enemies
- **Despawn**: Nếu vượt quá boundary hoặc hit enemy
- **Pooled**: Reuse từ pool thay vì destroy

#### Spread Pattern
| Level | Amount | Spread |
|-------|--------|--------|
| 1 | 1 | Không spread |
| 2+ | 2-3 | Fan spread |

#### Upgrade Progression
- Level 1: 1 bullet, base damage
- Level 2-5: Tăng damage + speed
- Level 6+: Add bullets (2-3 spread)
- Level 15+: Max spread + max damage

---

## UI & HUD

### 1. Health Slider
- **Position**: Top-left corner
- **Display**: Current HP / Max HP
- **Color**: Đỏ (cảnh báo khi < 30%)
- **Visual**: Slider bar + Text number

### 2. Energy Slider
- **Position**: Top-left corner (dưới Health)
- **Display**: Current Energy / Max Energy
- **Color**: Xanh dương (boost ready), vàng (low energy)
- **Visual**: Slider bar + Text number

### 3. Experience Slider
- **Position**: Bottom-left corner
- **Display**: Current EXP / EXP Needed for Next Level
- **Color**: Vàng/Xanh (progression indicator)
- **Visual**: Slider bar + Text number + Level indicator

### 4. Pause Panel
- **Trigger**: ESC hoặc GamePad Button
- **Contains**: 
  - Resume button
  - Main Menu button
  - Quit Game button
- **Effect**: Time.timeScale = 0 (pause toàn bộ)

### 5. Game Over Screen
- **Trigger**: Health <= 0
- **Delay**: 2 giây (chờ animation)
- **Contains**:
  - Game Over text
  - Final Score / Time Survived
  - New Game button
  - Main Menu button

---

## AUDIO

### 1. Sound Effects

| Sound ID | Trigger | Loại | Duration |
|----------|---------|------|----------|
| **shoot** | Phaser fires | Weapon | 0.1s |
| **hit** | Player takes damage | Impact | 0.1s |
| **fire** | Boost activated | Mechanic | 0.2s |
| **ice** | Player dies | Major | 0.3s |
| **boom2** | Asteroid destroyed | Explosion | 0.2s |
| **boom3** | Boss destroyed | Explosion | 0.5s |
| **hitRock** | Bullet hits asteroid | Impact | 0.1s |
| **hitArmor** | Bullet hits boss | Impact | 0.15s |
| **bossCharge** | Boss enters charge | Signal | 0.2s |
| **bossSpawn** | Boss appears | Major | 0.3s |
| **locustCharge** | Locust charges | Signal | 0.15s |
| **beetleHit** | Locust hit | Impact | 0.1s |
| **beetleDestroy** | Locust destroyed | Destruction | 0.2s |
| **pause** | Game paused | UI | 0.1s |
| **unpause** | Game resumed | UI | 0.1s |

### 2. Audio Features
- **Pitch Variation**: `PlayModifiedSound()` → Random pitch 0.7-1.3
- **Volume**: TBD (master volume control)
- **Mixer Groups**: SFX vs Music (separate)

---

## INPUT SYSTEM

### Keyboard Controls
| Input | Action | Loại |
|-------|--------|------|
| **W / ↑** | Move Up | Movement |
| **S / ↓** | Move Down | Movement |
| **A / ←** | Move Left | Movement |
| **D / →** | Move Right | Movement |
| **Space** | Hold: Boost / Release: Stop Boost | Mechanic |
| **Right Shift** | Shoot | Attack |
| **ESC** | Pause/Unpause | Menu |
| **P** | Pause (Alternative) | Menu |

### GamePad Controls (Unity Input Manager)
| Input | Mapping | Action |
|-------|---------|--------|
| **Left Stick** | Horizontal/Vertical | Movement |
| **Fire2 (RB)** | Space → Boost | Mechanic |
| **Fire1 (RT)** | Right Shift → Shoot | Attack |
| **Fire3 (Menu)** | ESC → Pause | Menu |

### Input Processing
```csharp
// Update (per frame)
float directionX = Input.GetAxisRaw("Horizontal");
float directionY = Input.GetAxisRaw("Vertical");
playerDirection = new Vector2(directionX, directionY).normalized;

// Fire2 = Boost
if (Input.GetKeyDown(KeyCode.Space)) EnterBoost();
if (Input.GetKeyUp(KeyCode.Space)) ExitBoost();

// Fire1 = Shoot
if (Input.GetKeyDown(KeyCode.RightShift)) {
    PhaserWeapon.Instance.Shoot();
}

// Pause
if (Input.GetKeyDown(KeyCode.Escape)) {
    GameManager.Instance.Pause();
}
```

---

## CAMERA & EFFECTS

### 1. Camera
- **Type**: Orthographic 2D
- **Position**: Fixed (0, 0, -10)
- **Size**: 10 units (aspect ratio 16:9 → 17.78 width)
- **Background**: Black (space)
- **No Follow**: Player tự do di chuyển, camera không follow

### 2. Visual Effects

#### Flash White Effect
- **Purpose**: Damage feedback
- **Duration**: 0.2s
- **Method**: Swap material to white temporarily
- **Applies to**: Player, Enemies, Asteroids

#### Explosion Particles
```
3 Pool sizes:
- Boom1Pool (Small): Player death
- Boom2Pool (Medium): Asteroid destruction
- Boom3Pool (Large): Boss destruction
```

#### Boost Particle Effect
- **ParticleSystem**: engineEffect
- **Active when**: Boost enabled
- **Stops when**: Boost disabled

#### Engine Glow
- Optional: Tint color when boosting

---

## LEVEL DESIGN

### 1. Level 1 (Duy nhất)

#### Layout
```
[Spawn Zone]        [Playable Area]        [Enemy Entry]
(-1 to 15, -5 to 5)                        (15, -5 to 5)
```

#### Difficulty Progression
```
Time 0-30s:   Easy (Asteroid waves)
Time 30-90s:  Medium (Mix Asteroid + Locust)
Time 90+s:    Hard (Frequent enemies + Boss spawns)
```

#### Boss Spawn Rule
- Mỗi 5 enemies thường tiêu diệt → Boss 1 spawn
- Boss 2 có thể spawn sau điều kiện TBD

#### Spawn Waves (Repeating)
```
Wave 0: Asteroid x10, interval 1.0s
Wave 1: Locustmorph x8, interval 1.5s
Wave 2: Asteroid x12 + Locust x5, interval 0.8s
Loop back to Wave 0...
```

### 2. Boundary Handling
| Object | Boundary | Behavior |
|--------|----------|----------|
| Player | None | Tự do di chuyển |
| Enemy | Left: x < -15 | SetActive(false) - Back to pool |
| Enemy | Top: y > 5 | Bounce (speedY *= -1) |
| Enemy | Bottom: y < -5 | Bounce |
| Asteroid | Similar | Similar |
| Bullet | Left: x < -20 | SetActive(false) |
| Bullet | Right: x > 20 | SetActive(false) |

---

## WIN/LOSE CONDITIONS

### LOSE Condition
```
Player Health ≤ 0
    ↓
GameManager.GameOver()
    ↓
WaitForSeconds(2f) [Explosion animation]
    ↓
Load "GameOver" scene
```

### Game Over Screen Flow
1. **Wait**: 2 giây (explosion effect + music fade out)
2. **Display**: Game Over text + Final Stats (Time survived, Score)
3. **Options**:
   - New Game → Load "Level1"
   - Main Menu → Load "MainMenu"
   - Quit → Application.Quit()

### Score System (Optional)
```
Score = Time Survived (seconds) + 
        Enemies Killed × 10 + 
        Final Level × 100
```

### No Win Condition
Đây là **Endless Mode** - không có điểm kết thúc, chỉ survival cao nhất

---

## CORE LOOP

### Frame-by-Frame Breakdown

```
┌─ UPDATE PHASE
│  ├─ Input Processing (Movement, Shoot, Boost)
│  ├─ Player Movement (velocity update)
│  ├─ Animator Update (animation blending)
│  └─ UI Update (sliders)
│
├─ FIXED UPDATE PHASE
│  ├─ Physics Simulation (collisions, velocities)
│  ├─ Enemy Movement (speedX, speedY)
│  ├─ Bullet Movement
│  └─ Collision Detection
│
├─ SPAWNING PHASE
│  ├─ Wave Timer Countdown
│  ├─ Check if Time to Spawn
│  ├─ Get Object from Pool
│  ├─ Set Position (random Y)
│  └─ Activate
│
├─ GAMEPLAY PHASE
│  ├─ Enemy Update (state machine)
│  ├─ Boss AI (Charge/Patrol)
│  ├─ Player Collision → Health update
│  ├─ Bullet Collision → Enemy damage
│  └─ Enemy Death → EXP drop
│
├─ PROGRESSION PHASE
│  ├─ EXP Check
│  ├─ Level Up? → Health++, Weapon++
│  └─ UI Refresh
│
└─ END FRAME
   ├─ Check Game Over (Health <= 0)?
   ├─ Trigger Game Over Screen?
   └─ Next Frame...
```

---

## TECHNICAL ARCHITECTURE

### Class Hierarchy
```
MonoBehaviour
├── GameManager (Singleton)
├── UIController (Singleton)
├── AudioManager (Singleton)
├── PlayerController (Singleton)
│   ├─ [Uses] Rigidbody2D, Animator
│   ├─ [Uses] FlashWhiite
│   └─ [Uses] PhaserWeapon
│
├── PhaserWeapon (Extends Weapon, Singleton)
│   ├─ [Has] List<WeaponStats>
│   └─ [Uses] ObjectPooler (bullets)
│
├── Enemy (Abstract)
│   ├─ Boss1 (Extends Enemy)
│   ├─ Boss2 (Extends Enemy)
│   └─ Locustmorph (Extends Enemy)
│       └─ [Has] List<Frames>
│
├── Asteroid
│   ├─ [Uses] Rigidbody2D
│   └─ [Uses] FlashWhiite
│
├── ObjectSpawners
│   ├─ [Has] List<Wave>
│   └─ [Uses] ObjectPooler
│
├── ObjectPooler
│   └─ [Has] List<GameObject> pool
│
├── FlashWhiite (Utility)
│   └─ [Coroutine] ResetMaterial()
│
├── FaceMovementDirection (Utility)
│   └─ [Rotate] Towards movement direction
│
└── MenuManager
    └─ Scene management
```

### Data Structures
```csharp
// Wave Definition
[System.Serializable]
public class Wave {
    public ObjectPooler pool;
    public float spawnTimer;
    public float spawnInterval;
    public int objectsPerWave;
    public int spawnedObjectsCount;
}

// Weapon Stats
[System.Serializable]
public class WeaponStats {
    public float speed;
    public int damage;
    public float size;
    public float amount;
    public float range;
}

// Enemy Frames (Variants)
[System.Serializable]
private class Frames {
    public Sprite[] sprites;
}
```

### Singleton Pattern Usage
```csharp
// Example: GameManager
public static GameManager Instance;

void Awake() {
    if (Instance != null) {
        Destroy(gameObject);
    } else {
        Instance = this;
    }
}

// Access anywhere:
GameManager.Instance.SetWorldSpeed(7f);
```

### Object Pooling Architecture
```csharp
// Bullet Pool (200 objects)
[SerializeField] ObjectPooler bulletPool = new ObjectPooler();
bulletPool.prefab = bulletPrefab;
bulletPool.poolSize = 200;

// Reuse:
GameObject bullet = bulletPool.GetPooledObject();
bullet.SetActive(true);

// When done:
bullet.SetActive(false); // Returns to pool
```

---

## SUMMARY

| Metric | Value |
|--------|-------|
| **Total Classes** | 15+ |
| **Total Scenes** | 3 (Menu, Level1, GameOver) |
| **Object Pools** | 7 |
| **Enemies** | 4 types (Boss1, Boss2, Locustmorph, Asteroid) |
| **Weapon Levels** | 20+ |
| **Max Player Level** | 30 |
| **Max Health** | 34 (5 + 29 levelups) |
| **Waves** | 3-5 repeating |
| **Game Duration** | Unlimited (Endless) |

---

**GDD Version**: 2.0  
**Last Updated**: June 2026  
**Status**: Complete & In Development
