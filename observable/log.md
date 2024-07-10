# log配置
```yaml
log:
  dir: "logs"
  name: "bluebird.log"
  level: "debug"
  max_size: 20      #一个文件最大可达 20M
  max_backups: 5   # 最多同时保存 5 个文件
  max_age: 10     # 一个文件最多可以保存 10 天
  compress: true  # 用 gzip 压缩
``` 

# log初始化
```go
func Init() error {
	logInfo := conf.GetLogInfo()
	if err := os.MkdirAll(logInfo.Dir, 0777); err != nil {
		return err
	}

	fileName := path.Join(logInfo.Dir, logInfo.Name)
	if _, err := os.Stat(fileName); err != nil {
		if _, err := os.Create(fileName); err != nil {
			return err
		}
	}

	logger := hertzzap.NewLogger(
		hertzzap.WithCoreEnc(
			zapcore.NewJSONEncoder(zapcore.EncoderConfig{
				MessageKey: "msg",
				LevelKey:   "level",
				NameKey:    "name",
				TimeKey:    "ts",
				CallerKey:  "caller",
				//FunctionKey:    "func",
				StacktraceKey:  "stacktrace",
				LineEnding:     "\n",
				EncodeTime:     zapcore.TimeEncoderOfLayout("2006-01-02 15:04:05.000"),
				EncodeLevel:    zapcore.LowercaseLevelEncoder,
				EncodeDuration: zapcore.SecondsDurationEncoder,
				EncodeCaller:   zapcore.ShortCallerEncoder,
			})),
		hertzzap.WithZapOptions(
			zap.AddCaller(),
			zap.AddCallerSkip(3)),
		hertzzap.WithExtraKeys(
			[]hertzzap.ExtraKey{
				consts.LogIDExtraKey,
			}))
	// 提供压缩和删除
	lumberjackLogger := &lumberjack.Logger{
		Filename:   fileName,
		MaxSize:    logInfo.MaxSize,    // 一个文件最大可达多少M。
		MaxBackups: logInfo.MaxBackups, // 最多同时保存多少个文件。
		MaxAge:     logInfo.MaxAge,     // 一个文件最多可以保存多少天。
		Compress:   logInfo.Compress,   // 是否用 gzip 压缩。
	}

	logger.SetOutput(lumberjackLogger)
	switch logInfo.Level {
	case "debug":
		logger.SetLevel(hlog.LevelDebug)
	case "info":
		logger.SetLevel(hlog.LevelInfo)
	case "warn":
		logger.SetLevel(hlog.LevelWarn)
	case "error":
		logger.SetLevel(hlog.LevelError)
	case "fatal":
		logger.SetLevel(hlog.LevelFatal)
	}
	hlog.SetLogger(logger)

	return nil
}
```

全局hlog直接调用即可