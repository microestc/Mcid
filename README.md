# Mcid
主键生成

using System;
using System.Threading;

namespace Microestc
{
    public class Mcid
    {
        private const int MaxId = 9999;//确保Millisecond级生成速度来设置最大值，这里每毫秒最多产生10000个id
        private const int MinId = 1;
        private static bool _new = true;
        private static int _incrementId = MinId;
        private static readonly DateTime _origin = new DateTime(2019, 1, 1);
        private static long _current = (long)(DateTime.UtcNow - _origin).TotalMilliseconds;
        private static readonly object _lock = new object();

        public static string NewMcid(string prefix = "")
        {
            lock (_lock)
            {
                if (_new)
                {
                    _new = false;
                    return $"{prefix}{_incrementId.ToString($"D{MaxId.ToString().Length}")}{_current}";
                }
                var timeStamp = (long)(DateTime.UtcNow - _origin).TotalMilliseconds;
                if (_current == timeStamp)
                {
                    _incrementId++;
                    if (_incrementId > MaxId)//考虑用Thread.Sleep(1),这里会在99停几毫秒时间，求更好的方式
                    {
                        return Loop(prefix);
                    }
                    return $"{prefix}{_incrementId.ToString($"D{MaxId.ToString().Length}")}{_current}";
                }

                _incrementId = MinId;
                _current = timeStamp;
                return $"{prefix}{_incrementId.ToString($"D{MaxId.ToString().Length}")}{_current}";
            }
        }

        private static string Loop(string prefix)
        {
            Thread.Sleep(1);
            var timeStamp = (long)(DateTime.UtcNow - _origin).TotalMilliseconds;
            if (_current == timeStamp)
            {
                return Loop(prefix);
            }

            _incrementId = MinId;
            _current = timeStamp;
            return $"{prefix}{_incrementId.ToString($"D{MaxId.ToString().Length}")}{_current}";
        }

    }
}




