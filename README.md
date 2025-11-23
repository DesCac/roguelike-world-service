# World Service (Go)

**Level initialization, NPC AI, spawn configuration** для roguelike игры.

## Ответственность

✅ **ЧТО ДЕЛАЕТ:**
- Level initialization (вызывает Map Generator)
- NPC AI Ticker (aggressive goblin logic)
- Monster spawn configuration
- Level completion check
- Town spawning для level 10
- Публикация `npc.action` событий в NATS

❌ **ЧТО НЕ ДЕЛАЕТ:**
- Генерация карты (делегирует Map Generator)
- Movement validation (делегирует Movement Service)
- Combat validation (делегирует Combat Service)

## Технологии

- **Язык:** Go 1.21+
- **gRPC:** google.golang.org/grpc
- **Event Bus:** NATS JetStream
- **Database:** PostgreSQL (для level templates)
- **Протоколы:** contracts/proto/world/

## Размер кода

**Target:** ~1000 LOC

## gRPC Contract

```protobuf
service WorldService {
  rpc InitializeLevel(InitializeLevelRequest) returns (InitializeLevelResponse);
  rpc CheckLevelComplete(CheckLevelCompleteRequest) returns (CheckLevelCompleteResponse);
  rpc GetNPCActions(GetNPCActionsRequest) returns (GetNPCActionsResponse);
}
```

## Запуск локально

```bash
# Install dependencies
go mod download

# Run service
go run main.go

# Health check
grpcurl -plaintext localhost:50053 grpc.health.v1.Health/Check
```

## Docker

```bash
# Build
docker build -t world-service:dev -f Dockerfile.dev .

# Run
docker run -p 50053:50053 \
  --env NATS_URL=nats://nats:4222 \
  --env DATABASE_URL=postgresql://game:game123@postgres:5432/game_central \
  world-service:dev
```

## Environment Variables

```bash
GRPC_PORT=50053
NATS_URL=nats://localhost:4222
DATABASE_URL=postgresql://game:game123@localhost:5432/game_central
LOG_LEVEL=info
```

## Testing

```bash
go test ./...
```

## Примеры использования

### Инициализация уровня
```json
Request:
{
  "game_id": "game_123",
  "level_number": 1,
  "player_ids": ["player_1", "player_2"]
}

Response:
{
  "initial_state": {
    "game_id": "game_123",
    "level_number": 1,
    "players": [...],
    "monsters": [...],
    "exit_position": {"x": 45, "y": 45}
  },
  "map_data": "..."
}
```

### NPC AI Tick
```json
Request:
{
  "game_id": "game_123",
  "level_number": 1,
  "monsters": [
    {
      "id": "goblin_1",
      "type": "goblin",
      "position": {"x": 30, "y": 24},
      "is_aggressive": true
    }
  ],
  "players": [
    {
      "id": "player_1",
      "position": {"x": 30, "y": 25}
    }
  ]
}

Response:
{
  "actions": [
    {
      "monster_id": "goblin_1",
      "action_type": "ATTACK",
      "target_player_id": "player_1"
    }
  ]
}
```

## Event Publishing

После NPC AI tick публикует:
```json
Topic: "npc.action"
Payload: {
  "game_id": "game_123",
  "monster_id": "goblin_1",
  "action_type": "attack",
  "target_player_id": "player_1"
}
```

## Performance

- **InitializeLevel:** < 50ms (с вызовом Map Generator)
- **GetNPCActions:** < 10ms
- **AI Tick rate:** 1 раз в секунду

---

**Статус:** Skeleton (требует имплементации)
**Следующие шаги:** Реализовать InitializeLevel, NPC AI Ticker, Level completion check
