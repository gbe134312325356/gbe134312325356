
#if defined(MOZ_CRASHREPORTER)
/minidump-analyzer
#endif
/pingsender
/pk12util
/ssltunnel
/xpcshell Push "crashreporter.exe"
  Push "default-browser-agent.exe"
  Push "minidump-analyzer.exe"
  Push "pingsender.exe"
  Push "updater.exe"
  Push "mozwer.dll"
  Push "${FileMainEXE}" "{product}/{product}",
        "{product}/{product}-bin",
        "{product}/minidump-analyzer",
        "{product}/pingsender",
        "{product}/plugin-container",
        "{product}/updater",
        "{product}/glxtest","{product}-bin",
            "*.dylib",
            "minidump-analyzer",
            "pingsender",
            "plugin-container.app/Contents/MacOS/plugin-container",
            "updater.app/Contents/MacOS/net.waterfox.updater",
            # 'xpcshell',
  if (AppConstants.platform === "android") {
      throw Components.Exception("", Cr.NS_ERROR_NOT_IMPLEMENTED);
    }

    let suppressPingsender = Services.prefs.getBoolPref(
      "toolkit.telemetry.testing.suppressPingsender",
      false
    );
    if (suppressPingsender) {
      this._log.trace("Silently skipping pingsender call in automation");
      return;
    }

    // By default, invoke `pingsender[.exe] URL path ...`.
    let exeName =
      AppConstants.platform === "win" ? "pingsender.exe" : "pingsender";
    let params = [];

    if (lazy.NimbusFeatures.pingsender.getVariable("backgroundTaskEnabled")) {
      // If using pingsender background task, invoke `firefox[.exe] --backgroundtask pingsender URL path ...`.
      exeName =
        AppConstants.MOZ_APP_NAME +
        (AppConstants.platform === "win" ? ".exe" : "");
      params = ["--backgroundtask", "pingsender"];
    }

    this._log.info(
      `Invoking '${exeName}${params.length ? " " + params.join(" ") : ""} ...'`
    );

    let exe = Services.dirsvc.get("GreBinD", Ci.nsIFile);
    exe.append(exeName);

    params.push(...pings.flatMap(ping => [ping.url, ping.path]));
    let process = Cc["@mozilla.org/process/util;1"].createInstance(
      Ci.nsIProcess
    );
    process.init(exe);
    process.startHidden = true;
    process.noShell = true;
    process.runAsync(params, params.length, observer);
    throw Components.Exception("", Cr.NS_ERROR_NOT_IMPLEMENTED);
