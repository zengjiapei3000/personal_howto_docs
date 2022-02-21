# add-geosites-on-advanced_route_settings

Sometimes, we need extend the geosites in settings about "advanced route settings".

## reference
1. qv2
2. domain-list-community

## my operations
1. Add the `dlc.dat` in `/usr/share/v2ray`.
2. open the 'settings' -> 'Advanced route settings', add the rules, if in `dlc.dat`, like `"ext:flag:tag"`;
  or you can relace the `/usr/share/geosite.dat`, the you can custom the geosite tags by youself, and add rules like "`geosite:tag`", such as `geosite:apple`.