# Kaorios Toolbox Guide

## Step 1: Download & place system files

- **Download** `KaoriosToolbox.apk`, `privapp_whitelist_com.kousei.kaorios.xml`, and `classes.dex` from:  
  https://github.com/Wuang26/Kaorios-Toolbox/releases
- **Add app directory:**
  - Copy the **KaoriosToolbox** folder to: `/system/priv-app/`
- **Copy files:**
  - Copy `privapp_whitelist_com.kousei.kaorios.xml` → `/system/etc/permissions/`
- **Permissions:**
  - Directories: `0755` (e.g., `KaoriosToolbox`, `lib`, `lib/*`)
  - Files: `0644` (for `.xml`, `.apk`, and any `.so`)

---

## Step 2: Add to `system/build.prop`

```properties
persist.sys.kaorios=kousei
```

---

## Step 3: Import `classes.dex`

Import **classes.dex** into the **last classes** of `framework.jar` (append as the last `classes*.dex`).

---

## Step 4: Patch `framework.jar` classes

> **Note:** If you are unsure about the exact injection spots, please refer to the **sample .smali templates** in the repo’s `Toolbox-docs/Template` folder.

### Class: `ApplicationPackageManager`

#### Method: `hasSystemFeature(Ljava/lang/String;)Z`

Find:
```
return v0
.end method
```

Add **above**:
```
invoke-static {v0, p1}, Lcom/android/internal/util/kaorios/ToolboxUtils;->KaoriosAttestationBL(ZLjava/lang/String;)Z
move-result v0
```

---

#### Method: `hasSystemFeature(Ljava/lang/String;I)Z`

Find:
```
return v0
.end method
```

Add **above**:
```
invoke-static {p1, p2, v0}, Lcom/android/internal/util/kaorios/ToolboxUtils;->KaoriosFeatures(Ljava/lang/String;IZ)Z
move-result v0
```

---

### Class: `Instrumentation`

#### Method: `newApplication(Ljava/lang/Class;Landroid/content/Context;)Landroid/app/Application;`

Find:
```
return-object v0
.end method
```

Add **above**:
```
invoke-static {p1}, Lcom/android/internal/util/kaorios/ToolboxUtils;->KaoriosProps(Landroid/content/Context;)V
```

---

#### Method: `newApplication(Ljava/lang/ClassLoader;Ljava/lang/String;Landroid/content/Context;)Landroid/app/Application;`

Find:
```
return-object v0
.end method
```

Add **above**:
```
invoke-static {p3}, Lcom/android/internal/util/kaorios/ToolboxUtils;->KaoriosProps(Landroid/content/Context;)V
```

---

### Class: `KeyStore2`

#### Method: `getKeyEntry(Landroid/system/keystore2/KeyDescriptor;)Landroid/system/keystore2/KeyEntryResponse;`

Find:
```
return-object v0
.end method
```

Add **above**:
```
invoke-static {v0}, Lcom/android/internal/util/kaorios/ToolboxUtils;->KaoriosKeybox(Landroid/system/keystore2/KeyEntryResponse;)Landroid/system/keystore2/KeyEntryResponse;
move-result-object v0
```

---

### Class: `AndroidKeyStoreSpi`

#### Method: `engineGetCertificateChain(Ljava/lang/String;)[Ljava/security/cert/Certificate;`

Add **below** `.registers XX`:
```
invoke-static {}, Lcom/android/internal/util/kaorios/ToolboxUtils;->KaoriosPropsEngineGetCertificateChain()V
```
