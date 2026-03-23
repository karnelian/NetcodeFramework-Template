# 네트워킹 패턴 코드 레퍼런스

> 개념 설명은 `SKILL.md` 참조. 이 문서는 실전 구현 코드를 다룬다.

---

## 1. 상태 보간 (Entity Interpolation)

```csharp
public class NetworkTransform
{
    private struct State
    {
        public float Timestamp;
        public Vector3 Position;
        public Quaternion Rotation;
    }

    private State[] _buffer = new State[32];
    private int _bufferIndex = 0;

    public void ReceiveState(float timestamp, Vector3 position, Quaternion rotation)
    {
        _buffer[_bufferIndex] = new State
        {
            Timestamp = timestamp,
            Position = position,
            Rotation = rotation
        };
        _bufferIndex = (_bufferIndex + 1) % _buffer.Length;
    }

    public (Vector3 pos, Quaternion rot) Interpolate(float renderTime)
    {
        State from = default, to = default;

        for (int i = 0; i < _buffer.Length; i++)
        {
            if (_buffer[i].Timestamp <= renderTime)
                from = _buffer[i];
            else
            {
                to = _buffer[i];
                break;
            }
        }

        if (from.Timestamp == 0 || to.Timestamp == 0)
            return (from.Position, from.Rotation);

        float t = Mathf.Clamp01(
            (renderTime - from.Timestamp) / (to.Timestamp - from.Timestamp));

        return (
            Vector3.Lerp(from.Position, to.Position, t),
            Quaternion.Slerp(from.Rotation, to.Rotation, t)
        );
    }
}
```

---

## 2. 클라이언트 예측 + 리콘실리에이션

```csharp
public class PredictivePlayer : MonoBehaviour
{
    private struct InputState
    {
        public int SequenceNumber;
        public float Timestamp;
        public Vector3 Movement;
    }

    private Queue<InputState> _pendingInputs = new();
    private int _sequenceNumber = 0;
    private Vector3 _predictedPosition;
    [SerializeField] private float moveSpeed = 10f;

    void Update()
    {
        Vector3 movement = new Vector3(
            Input.GetAxis("Horizontal"), 0, Input.GetAxis("Vertical")
        ) * moveSpeed * Time.deltaTime;

        var input = new InputState
        {
            SequenceNumber = _sequenceNumber++,
            Timestamp = Time.time,
            Movement = movement
        };

        SendInputToServer(input);

        // 로컬 예측
        _predictedPosition += movement;
        transform.position = _predictedPosition;

        _pendingInputs.Enqueue(input);
    }

    public void ReceiveServerState(int lastProcessedInput, Vector3 serverPosition)
    {
        // 확인된 입력 제거
        while (_pendingInputs.Count > 0 &&
               _pendingInputs.Peek().SequenceNumber <= lastProcessedInput)
        {
            _pendingInputs.Dequeue();
        }

        // 서버 위치에서 시작
        _predictedPosition = serverPosition;

        // 남은 입력 재적용 (리콘실리에이션)
        foreach (var input in _pendingInputs)
            _predictedPosition += input.Movement;

        // 보정이 크면 스냅
        if (Vector3.Distance(transform.position, _predictedPosition) > 0.1f)
            transform.position = _predictedPosition;
    }

    private void SendInputToServer(InputState input) { /* 네트워크 전송 */ }
}
```

---

## 3. 래그 보상 (서버 사이드 리와인드)

```csharp
public class LagCompensation
{
    private struct HistoricalState
    {
        public float Timestamp;
        public Vector3 Position;
        public Bounds Hitbox;
    }

    private Dictionary<int, Queue<HistoricalState>> _history = new();
    private const float MaxHistoryTime = 1.0f; // 1초

    public void RecordState(int playerId, Vector3 position, Bounds hitbox)
    {
        if (!_history.ContainsKey(playerId))
            _history[playerId] = new Queue<HistoricalState>();

        var queue = _history[playerId];
        queue.Enqueue(new HistoricalState
        {
            Timestamp = Time.time,
            Position = position,
            Hitbox = hitbox
        });

        // 오래된 상태 제거
        while (queue.Count > 0 && Time.time - queue.Peek().Timestamp > MaxHistoryTime)
            queue.Dequeue();
    }

    public bool ProcessHitscan(int shooterId, float clientTimestamp, Ray ray, out int hitPlayerId)
    {
        foreach (var kvp in _history)
        {
            if (kvp.Key == shooterId) continue;

            HistoricalState state = GetStateAtTime(kvp.Value, clientTimestamp);
            if (state.Hitbox.IntersectRay(ray))
            {
                hitPlayerId = kvp.Key;
                return true;
            }
        }
        hitPlayerId = -1;
        return false;
    }

    private HistoricalState GetStateAtTime(Queue<HistoricalState> history, float targetTime)
    {
        HistoricalState closest = default;
        float minDelta = float.MaxValue;
        foreach (var state in history)
        {
            float delta = Mathf.Abs(state.Timestamp - targetTime);
            if (delta < minDelta) { minDelta = delta; closest = state; }
        }
        return closest;
    }
}
```

---

## 4. 메시지 직렬화 (바이너리)

```csharp
public class NetworkWriter
{
    private MemoryStream _stream = new MemoryStream();
    private BinaryWriter _writer;

    public NetworkWriter() { _writer = new BinaryWriter(_stream); }

    public void WriteInt(int v) => _writer.Write(v);
    public void WriteFloat(float v) => _writer.Write(v);
    public void WriteBool(bool v) => _writer.Write(v);

    public void WriteVector3(Vector3 v)
    {
        _writer.Write(v.x); _writer.Write(v.y); _writer.Write(v.z);
    }

    // 압축 Vector3 (16-bit per component)
    public void WriteVector3Compressed(Vector3 v, float min, float max)
    {
        _writer.Write(CompressFloat(v.x, min, max));
        _writer.Write(CompressFloat(v.y, min, max));
        _writer.Write(CompressFloat(v.z, min, max));
    }

    private ushort CompressFloat(float value, float min, float max)
    {
        float normalized = Mathf.Clamp01((value - min) / (max - min));
        return (ushort)(normalized * ushort.MaxValue);
    }

    public byte[] ToArray() => _stream.ToArray();
}

public class NetworkReader
{
    private BinaryReader _reader;

    public NetworkReader(byte[] data)
    {
        _reader = new BinaryReader(new MemoryStream(data));
    }

    public int ReadInt() => _reader.ReadInt32();
    public float ReadFloat() => _reader.ReadSingle();

    public Vector3 ReadVector3() => new(
        _reader.ReadSingle(), _reader.ReadSingle(), _reader.ReadSingle());

    public Vector3 ReadVector3Compressed(float min, float max) => new(
        DecompressFloat(_reader.ReadUInt16(), min, max),
        DecompressFloat(_reader.ReadUInt16(), min, max),
        DecompressFloat(_reader.ReadUInt16(), min, max));

    private float DecompressFloat(ushort value, float min, float max)
    {
        return min + (value / (float)ushort.MaxValue) * (max - min);
    }
}
```

---

## 5. Interest 관리 (Relevancy)

```csharp
public class InterestManager
{
    private Dictionary<int, Vector3> _positions = new();
    private float _relevancyRadius = 100f;

    public HashSet<int> GetRelevantPlayers(int playerId)
    {
        if (!_positions.TryGetValue(playerId, out Vector3 playerPos))
            return new HashSet<int>();

        var relevant = new HashSet<int>();
        foreach (var kvp in _positions)
        {
            if (kvp.Key == playerId) continue;
            if (Vector3.Distance(playerPos, kvp.Value) <= _relevancyRadius)
                relevant.Add(kvp.Key);
        }
        return relevant;
    }

    public void BroadcastToRelevant(int senderId, byte[] message)
    {
        foreach (int id in GetRelevantPlayers(senderId))
            SendMessage(id, message);
    }

    private void SendMessage(int id, byte[] msg) { /* 전송 */ }
}
```

---

## 6. 델타 압축

```csharp
public class DeltaCompressor
{
    private Dictionary<int, NetworkPlayer> _lastSent = new();

    public byte[] CompressState(NetworkPlayer current)
    {
        if (!_lastSent.TryGetValue(current.PlayerId, out var previous))
            return SerializeFullState(current);

        var writer = new NetworkWriter();
        byte flags = 0;

        if (Vector3.Distance(current.Position, previous.Position) > 0.01f)
        {
            flags |= 1 << 0;
            writer.WriteVector3Compressed(current.Position, -1000f, 1000f);
        }

        if (Quaternion.Angle(current.Rotation, previous.Rotation) > 1f)
        {
            flags |= 1 << 1;
            // WriteQuaternionCompressed 구현 필요
        }

        if (Mathf.Abs(current.Health - previous.Health) > 0.1f)
        {
            flags |= 1 << 2;
            writer.WriteFloat(current.Health);
        }

        byte[] data = writer.ToArray();
        byte[] result = new byte[data.Length + 1];
        result[0] = flags;
        Array.Copy(data, 0, result, 1, data.Length);

        _lastSent[current.PlayerId] = current;
        return result;
    }

    private byte[] SerializeFullState(NetworkPlayer p) { /* 전체 직렬화 */ return null; }
}
```
