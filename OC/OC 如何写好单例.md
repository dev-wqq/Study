@interface DWSingletonManager: NSObject <NSCopying>

+ (instancetype)sharedInstance;
- (void)reset;

@end

static DWSingletonManager *_sharedInstance;
static dispatch_once_t _onceToken;

@implementation DWSingletonManager

+ (instancetype)sharedInstance {
    if (_sharedInstance) return _sharedInstance;
    return [[self alloc] init];
}

+ (instancetype)allocWithZone:(struct _NSZone *)zone {
    dispatch_once(&_onceToken, ^{
        _sharedInstance = [super allocWithZone:zone];
        // Initialization property
    });
    return _sharedInstance;
}

- (id)copyWithZone:(NSZone *)zone {
    return _sharedInstance;
}

- (void)reset {
    _onceToken = 0;
    _sharedInstance = nil;
}
@end


