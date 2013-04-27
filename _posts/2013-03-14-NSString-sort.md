##NSString 的比较排序在，Cocoa中有如下几种方式

###1. 使用使用数组的sortedArrayUsingComparator 再结合NSString compare 之类的方法，可以指定Option

    -(NSArray *)sortedArrayUsingComparator:(NSComparator)cmptr NS_AVAILABLE(10_6, 4_0);

    - (NSArray *)sortedArrayWithOptions:(NSSortOptions)opts usingComparator:(NSComparator)cmptr NS_AVAILABLE(10_6, 4_0);

    - (NSComparisonResult)compare:(NSString *)string;

    - (NSComparisonResult)compare:(NSString *)string options:(NSStringCompareOptions)mask;

    - (NSComparisonResult)compare:(NSString *)string options:(NSStringCompareOptions)mask range:(NSRange)compareRange;

    - (NSComparisonResult)compare:(NSString *)string options:(NSStringCompareOptions)mask range:(NSRange)compareRange locale:(id)locale; 

###option 是比较的规则

    enum {

       NSCaseInsensitiveSearch = 1, //忽略大小

       NSLiteralSearch = 2,//

       NSBackwardsSearch = 4,//从后往前比较

       NSAnchoredSearch = 8,

       NSNumericSearch = 64,//数字和字母比较

       NSDiacriticInsensitiveSearch = 128,//忽略发音，比如中文的 妈（第一声），马（第三声）

       NSWidthInsensitiveSearch = 256, //忽略宽度

       NSForcedOrderingSearch = 512, //

       NSRegularExpressionSearch = 1024

    };


###2. 在数组中使用sort方法，再结合string的selector

    - (NSArray *)sortedArrayUsingFunction:(NSInteger (*)(id, id, void *))comparator context:(void *)context;

    - (NSArray *)sortedArrayUsingFunction:(NSInteger (*)(id, id, void *))comparator context:(void *)context hint:(NSData *)hint;

    - (NSArray *)sortedArrayUsingSelector:(SEL)comparator;

###NSString.h

    - (NSComparisonResult)caseInsensitiveCompare:(NSString *)string;

    - (NSComparisonResult)localizedCompare:(NSString *)string;

    - (NSComparisonResult)localizedCaseInsensitiveCompare:(NSString *)string;

    - (NSComparisonResult)localizedStandardCompare:(NSString *)string

###sortedArrayUsingFunction传递是参数是函数指针，就不一定是要某个类的selector

    NSInteger intSort(id num1, id num2, void *context)
    {
        int v1 = [num1 intValue];
        int v2 = [num2 intValue];
        if (v1 < v2)
            return NSOrderedAscending;
        else if (v1 > v2)
            return NSOrderedDescending;
        else
            return NSOrderedSame;
    }

###返回比较结果的NSInteger 可以对应

    enum {

       NSOrderedAscending = -1, //由小到达

       NSOrderedSame, //相等

       NSOrderedDescending //由大到小
    };

    typedef NSInteger NSComparisonResult;



###而使用sortedArrayUsingSelector 就可以使用到系统提供的自己个默认的slector

###3. 使用数组的descriptor

    - (NSArray *)sortedArrayUsingDescriptors:(NSArray *)sortDescriptors;    // returns a new array by sorting the objects of the receiver

    NSSortDescriptor* sortDesc = [NSSortDescriptor sortDescriptorWithKey:@"name" ascending:YES selector:@selector(localizedCaseInsensitiveCompare:)];


###而系统这些API底层是调用ICU库实现的， 而且结合locale的参数可以实现多种不同的比较

    static NSComparisonResult compareStringByPassingLocaleName(NSString *a, NSString *b, const char *localeName)
    {
	    NSLocale *locale = [[NSLocale alloc] initWithLocaleIdentifier:[NSString stringWithUTF8String:localeName]];
	    NSComparisonResult result = [a compare:b options:0 range:NSMakeRange(0, [a length]) locale:locale];
	    [locale release];
	    return result;
    }

    #define COMPARE(x) ((NSComparisonResult)compareStringByPassingLocaleName(self, anotherString, x))
    - (NSComparisonResult)compareChineseByStrokeOrder:(NSString *)anotherString
    {
	return COMPARE("zh@collation=stroke");
    }
    - (NSComparisonResult)compareChineseByPinyinOrder:(NSString *)anotherString
    {
	return COMPARE("zh@collation=pinyin");
    }
    - (NSComparisonResult)compareChineseByBIG5Order:(NSString *)anotherString
    {
	return COMPARE("zh@collation=big5han");
    }
    - (NSComparisonResult)compareChineseByGB2312Order:(NSString *)anotherString
    {
	return COMPARE("zh@collation=gb2312");
    }
    - (NSComparisonResult)compareChineseByRadicalOrder:(NSString *)anotherString
    {
	return COMPARE("zh@collation=unihan");
    }
    #undef COMPARE
    

###[详细代码参见](https://github.com/zonble/NSString-CustomCompare/blob/master/NSString%2BCustomCompare/NSString%2BCustomCompare.mm)


