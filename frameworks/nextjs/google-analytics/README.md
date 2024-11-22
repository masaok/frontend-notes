### ga4.ts

```ts
import ga4 from "react-ga4";

const TRACKING_ID = "G-XXXXXXXXXX";
const isProduction = process.env.NODE_ENV === "production";

export const init = () =>
  ga4.initialize(TRACKING_ID, {
    testMode: !isProduction,
  });

export const sendEvent = (name: string) =>
  ga4.event("screen_view", {
    app_name: "myApp",
    screen_name: name,
  });

export const sendPageview = (path: string) =>
  ga4.send({
    hitType: "pageview",
    page: path,
  });
```

### useAnalytics.ts

```ts
import React from "react";

import * as analytics from "./ga4";
import { usePathname } from "next/navigation";

export function useAnalytics() {
  const pathname = usePathname();

  React.useEffect(() => {
    analytics.init();
  }, []);

  React.useEffect(() => {
    analytics.sendPageview(pathname);
  }, [pathname]);
}

export default useAnalytics;
```

### Usage

```ts
export default function DashboardLayout({
  children,
}: Readonly<{
  children: React.ReactNode
}>) {
  useAnalytics()
  ...
```
