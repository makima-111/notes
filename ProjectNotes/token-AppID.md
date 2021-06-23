



token<->AppID

```
private static Integer tokenToAppId(String gameToken) {
    switch (gameToken) {
        case "51bcd0":
            return 100;
        case "d7a935":
            return 102;
        case "633b2d":
            return 110;
        case "ba3005":
            return 112;
        case "3ad472": // Bricks
            return 132;
        case "31dfe6": // Monster Lite
            return 142;
        case "89741e": // cake
            return 150;
        case "114151": // Alina
            return 160;
        case "7ee6c8": // Alina Other
            return 162;
        default:
            return 0;
    }
}

public static Integer tokenToAppId(String gameToken, String osName) {
    if (osName.equals("ios") || "iOS".equals(osName) || osName.equals(Device.IOS.getValue())) {
        return tokenToAppId(gameToken) + 1;
    } else {
        return tokenToAppId(gameToken) + 2;
    }
}
```