> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [forum.90sec.com](https://forum.90sec.com/t/topic/2225) ![](https://forum.90sec.com/letter_avatar_proxy/v4/letter/y/ed655f/45.png)

某多多的 ant-content 核心算法解密版本，耗时 4 天 3 夜，并非网上直接把代码扣下来的。完全解密版本

### 解密方法和代码

代码是用 ast 来解密的。利用 babel 处理，解密一部分 + 手动修复代码。  
AST 相关的教程和文档

```
https://steakenthusiast.github.io/
https://evilrecluse.top/Babel-traverse-api-doc/
https://astexplorer.net/
```

以下代码不适用于所有加密

```
const fs = require("fs");
const { parse } = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const generator = require("@babel/generator").default;
const t = require("@babel/types");
const beautify = require("js-beautify");


const jscode = fs.readFileSync("./pdd_ant2.js").toString("utf-8")

let ast = parse(jscode);

function node_eq(node, node2) {
    if (node2.type == "VariableDeclarator") {
        node2 = node2.init;
    }
    return node.start == node2.start && node.end == node2.end;
}

function var_get_referencePaths(nodePath) {
    const context = nodePath.parentPath.get("left").context;
    const bindings = context.scope.bindings[nodePath.parent.left.name];
    return bindings.referencePaths;
}
function get_scope_binding(path) {
    if (["callee", "MemberExpression"].includes(path.key) || path.get("object").node) {
        return path.scope.getBinding(path.get("object").node.name)
    } else if (path.node.name) {
        return path.scope.getBinding(path.node.name)
    } else if (path.get("id").node) {
        return path.scope.getBinding(path.get("id").node.name)
    } else {
        console.log("没有binding")
    }
}


//查找当前变量所有使用的地方
function relyon_var(referencePaths, codes = new Map()) {
    //作用域
    for (let i = 0; i < referencePaths.length; i++) {
        //使用的path
        const nodePath = referencePaths[i];
        //在当前方法内直接返回不管
        // if (node_eq(nodePath.node, node)) {
        //     continue;
        // }
        //说明是一个赋值操作 var n=u;
        if (["right"].includes(nodePath.key)) {
            //抽离关于n的代码
            const varScope = var_get_referencePaths(nodePath);
            relyon_var(varScope);
            // const id2 = nodePath.node.start + "_" + nodePath.node.end;
            // codes.set(id2, nodePath)
            // //查找当前nodepath使用的引用
            // const context = nodePath.parentPath.get("left").context;
            // const bindings = context.scope.bindings[nodePath.parent.left.name];
            // const referencePaths = bindings.referencePaths;
            // //引用
            // for (let index = 0; index < array.length; index++) {
            //     const element = array[index];

            // }
            // console.log(binds.referencePaths)
        }
    }
}


function encode1(arg1, arg2) {
    var u = ["fSohrCk0cG==", "W4FdMmotWRve", "W7bJWQ1CW6C=", "W5K6bCooW6i=", "dSkjW7tdRSoB", "jtxcUfRcRq==", "ALj2WQRdQG==", "W5BdSSkqWOKH", "lK07WPDy", "f8oSW6VcNrq=", "eSowCSkoaa==", "d8oGW7BcPIO=", "m0FcRCkEtq==", "qv3cOuJdVq==", "iMG5W5BcVa==", "W73dVCo6WPD2", "W6VdKmkOWO8w", "zueIB8oz", "CmkhWP0nW5W=", "W7ldLmkSWOfh", "W5FdIqdcJSkO", "aCkBpmoPyG==", "l27dICkgWRK=", "s05AWR7cTa==", "bttcNhdcUW==", "gJldK8kHFW==", "W5Sso8oXW4i=", "FgC0W7hcNmoqwa==", "xmkPhdDl", "e14kWRzQ", "BNFcVxpdPq==", "z1vadK0=", "W7yOiCk2WQ0=", "qLb7lg0=", "t8o6BwhcOq==", "gmk6lYD9WPdcHSoQqG==", "oqldGmkiCq==", "rmo+uKlcSW==", "dSoIWOVdQ8kC", "iXSUsNu=", "W5ipW4S7WRS=", "WPtcTvOCtG==", "A3CcAmoS", "lCotW6lcMba=", "iuGzWPLz", "WQVdPmoKeSkR", "W4ydoCkqWQ4=", "jCobW47cNXC=", "W4tdJCkNWOCJ", "hCo/W7ZcSJ8=", "BNuZW6NcMG==", "b8kFW6hdN8oN", "W4SpoCkXWQK=", "cXddOmkDFa==", "W63dHSoyWQft", "W6ldSmk0WRj4", "A2bHWOtcHeeMyq==", "f3VcSSk/xG==", "qg1u", "ftyivga=", "DCkhpsfe", "WR3cKmo3oMWEw8kK", "yev3", "W4xdMKSejbm=", "W797WOL7W4m=", "W6xdOCkKWQXw", "gcCUye0=", "W7WXkmomb8kT", "c8kIesD0", "WOTpEW==", "ySo3E8oVWPy=", "iNyhW5lcNLNcG8kYWQu=", "W7JdMSkfWRnD", "FfijW5tcHW==", "xCokW54Zzq==", "W77dUsi=", "W5FdHfa6eq==", "E1FcQvVdSG==", "eZ/dNCo4AG==", "CgPmWQZdKa==", "A8oLECoJWPS=", "oCoSW7VcTJC=", "mCoADa==", "W7DXuSouDq==", "ic3dQCo8ua==", "rN3cIa==", "W6/dJ8kPWRGQ", "W4xdLYlcPmkc", "F3JcPvZdLa==", "xCk8iHn4", "qg15", "W5/dL8oOWPr4", "hW41C3C=", "sSoZzwxcPW==", "ywdcUvNdUW==", "t0TzWQpdIG==", "lv7dJSoIjq==", "W5Tzxq==", "W6DnWQK=", "W5mGaCkFWRC=", "W6LmWO5+W6C=", "WR7dQmoJa8k+", "emkFW4ddOmob", "imk8imoNEa==", "W4ZdP8kaWPvc", "F8k4WO40W4e=", "cSoHE8k9cG==", "jw4TW5dcSW==", "wuJcOKRdTa==", "swNcQx/dGG==", "aCkSiCoMEq==", "W6pdS8owWQTH", "WRFdQmonjmkT", "cKBdGCkpWOm=", "oCoWW4VcPIa=", "WQddSSoUjmks", "c8kdW5JdM8oE", "W7b0AGvl", "sCk4WOylW60=", "nXNdSmkXvW==", "W67dRSkjWOqj", "W44EcCohW6O=", "W6ddPmkpWRHN", "W7tdVIVcOSkR", "qg3dVG==", "W7Ofcmofda==", "WRDmW5VcLq==", "CSoRW4W4Aq==", "mmo0WP3dVmkj", "i8omW6ZcPd8=", "CSkaWQyvW4m=", "ACkMWQCLW4q=", "W5pdOCk0WRv3", "W7yDW44SWP8=", "WRP8W5dcNmkd", "ymkNaID5", "cfeTWRT6", "W6WdbmkmWO0=", "eSo3WQldVCkU", "W5flwZrl", "WPVcTe4tWQu=", "DuCPumok", "hLpcKCksqXe=", "g3hdUCkoWRu=", "sL0sW6JcPW==", "lf7dL8oOpG==", "w8k4WPWJW7u=", "i08mW5dcUW==", "kb/dU8klsW==", "WOhcMSoW", "W5LnfG==", "F8kJWQmxW6m=", "W5ldU0CDca==", "eKRdKmkoWPG=", "tmouW60=", "gSkrW7JdVSor", "WPNcP8oc", "DhLAmLW=", "sSo0EfdcQq==", "W6ygW689WQq=", "W6CPimkIWQa=", "WRJdLmoynSkY", "W5iimCkDWRa=", "oMhdN8kPWRHV", "eNqQWQHn", "bmkakSoHW4u=", "W4PxEbvN", "WQhcQxSWyW==", "xCoKEW==", "guBcISk2yG==", "nviRW4BcSq==", "m3tcVmkXCJ9YWQyXd8kuWQfJW71fWPmnWRj+WR1tW6WbW4PDdCkrkLbDs8ozWR4gySoyv20rWO3dJJpdIh9DWPhcGCoctKFcN8kTW6nHvbLRkg9MeKhdHCoP", "W7iZfmolW4q=", "p1JdGSk4WPW=", "ns3cTuhcMSk6u8kj", "q8kmhr5p", "lWCxtKW=", "pmk+hSoYFG==", "bdFdKmkIwa==", "WR/cMSoL", "csCy", "W7BdKCkmWPfO", "tCkeWPyXW70=", "smkVWRK=", "dNFdQSokiq==", "W5OyoCoLW5O=", "W4RcIZ0xW5hdPCkaWPddO0aoE8oCwXVcSgbVtWbqW6u=", "iKNdK8khWRa=", "WQtdQCommSkg", "W6ddU8k1WQ94", "ASoXAMRcHG==", "gMhdKCoBna==", "eCk5mSoEW6K2v8octbK=", "pmo+Fmkfea==", "f3y8WPL0Ex4=", "oSkmm8oczq==", "W7ldK8oWWRnrW6WtqMG0W7/cMxbU", "W7uwdmofbG==", "A8oqyudcPG==", "s8oHt3FcTq==", "a8okBCkAdq==", "W7mvg3OI", "E8kLWR0dW7i=", "W78qhKSF", "W6XMWRHsW6K=", "hCoyzSk7fa==", "WQNcKSoHp1S=", "oCkaiCocW6i=", "bSoEW5ZcVXq=", "W5pdVCkHWRj3", "eehdNSoGhG==", "W4VdTmkhWRO=", "W73dMte=", "bqBcJelcTG==", "WOpcKLXWBa==", "W7uRa0OKnwpdRmoq", "WO3cKSoHW7C4", "WPRcOCofl0i=", "BxvOWPhcSa==", "hwK0W7tcJq==", "BMOjW5lcGq==", "cmouWONdUmk8", "E8k9WQyjW7NdNa==", "WRNcQSoFi0S=", "zLTHWPpcUW==", "WRPjW7BcLCkB", "BLRcLMddLW==", "s8kzWOiiW5m=", "W40mW4uqWP8=", "i13cMCk7Ea==", "WQBcLMupWOu=", "x8o2xmoD", "hCkBcCoLvW==", "FmkEWRShW5q=", "W58ikmo+W7K=", "W4KehmkSWOG=", "WQZcLCod", "WQtcHgXHCa==", "W4ldRbpcSmkY", "r8oKW5ukr0e+gW==", "dSkjW4FdLCoY", "cGa6Ee4=", "W69pymoVuW==", "WQRcSCo7i0i=", "W5RdICoWWQPaW70ode4=", "cfiNWODs", "W7rzWPr/W4u=", "ySkuecz+", "W4qsW70WWOq=", "W5VdS8kmWPXz", "W44jW7W=", "pxRcGW==", "ye5hngpdUa==", "WRRcQfT0va==", "WQxcImouW7CY", "qLRcJKddTa==", "p8o6q8kUdW==", "W4nlWRLvW6W=", "p3hdQ8kzWOe=", "W4eFeCojW5W=", "W43dNCoMWRG=", "nNCqW7lcQW==", "FCoqw3dcUq==", "W4BdGSkKWQ8+", "rmo8q1/cKW==", "D0assmov", "f0eQWODU", "nJXVfCo5W6VcVIniWPKKcCkpWO0fW63dNI4fWPziiSkWEmowWO12AKqNWQvPyCkMmb8aCConW7ddQCkmxs3cG3xdJuuMW7FdJCoqWQndsmk9WQzzW5mgWP/cUHmx", "pCoRymkabCoqta==", "i2xdImk+", "owFdVSkkWOm=", "WPNcK1H+Ca==", "W4FdKJxcICkP", "W4hdNSkuWO4=", "W7Gol8oAW6O=", "W61RWRrOW4y=", "W7qAn8ksWQK=", "WPVcRvWNWOG=", "xmoyrwFcQW==", "WOz7W4hcRSkB", "l1yQW5RcSW==", "zvJcQvZdNa==", "W4hdPSobWPvy", "nWldKCoIvG==", "CeTyh3K=", "pa/cVexcLG==", "cmk0W6JdUSoK", "AwSxW5ZcHq==", "jIpcKfdcOW==", "W5r5WQXpW74=", "n8k1mmoHW4G=", "xe4JW7FcMW==", "hmolw8kViW==", "gfutW6hcSG==", "hflcVSkzrW==", "jZpcRN/cRq==", "W7tdV8kF", "ig0UW7VcLW==", "b03dGCkBWP0=", "nYFcPW==", "W4ueW6StWP0=", "W4BdN8ogWR9D", "qe89qCo3", "W68dgmkSWR4=", "Ae0FsmoD", "pSoVECkojG==", "W6aplSoBfG==", "mq/dR8omya==", "amkMiCojW40=", "xN5GWPVcJa==", "W67dJmk4WQji", "fxRcVCk7yG==", "fSkLoSoLW7a=", "a8oCWPJdP8kt", "e8o0WRxdI8kv", "ChO3W6NcMa==", "awVdPmkGWO0=", "nCk0W6pdMCod", "W4xdP8kOWO5J", "lSowxSk0fW==", "js/cPwVcTW==", "WOJdRmo9amkt", "nsRcULdcUmkH", "gCkIW4FdLmoF", "DmovW7erzG==", "cSoFD8kfeq==", "WRVcH8ouW7aC", "WPvCW6xcKSkr", "W4qRW4arWQW=", "WPpcPgjfFW=="];
    var n = u;
    var e = 280;
    (function (t) {
        for (; --t;)
            n.push(n.shift())
    })(++e);
    var c = function t(n, r) {
        var e = u[n -= 0];
        void 0 === t.dkfVxK && (t.jRRxCS = function (t, n) {
            for (var r = [], e = 0, o = void 0, a = "", i = "", u = 0, c = (t = function (t) {
                for (var n, r, e = String(t).replace(/=+$/, ""), o = "", a = 0, i = 0; r = e.charAt(i++); ~r && (n = a % 4 ? 64 * n + r : r,
                    a++ % 4) ? o += String.fromCharCode(255 & n >> (-2 * a & 6)) : 0)
                    r = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/=".indexOf(r);
                return o
            }(t)).length; u < c; u++)
                i += "%" + ("00" + t.charCodeAt(u).toString(16)).slice(-2);
            t = decodeURIComponent(i);
            var W = void 0;
            for (W = 0; W < 256; W++)
                r[W] = W;
            for (W = 0; W < 256; W++)
                e = (e + r[W] + n.charCodeAt(W % n.length)) % 256,
                    o = r[W],
                    r[W] = r[e],
                    r[e] = o;
            W = 0,
                e = 0;
            for (var d = 0; d < t.length; d++)
                e = (e + r[W = (W + 1) % 256]) % 256,
                    o = r[W],
                    r[W] = r[e],
                    r[e] = o,
                    a += String.fromCharCode(t.charCodeAt(d) ^ r[(r[W] + r[e]) % 256]);
            return a
        }
            ,
            t.vDRBih = {},
            t.dkfVxK = !0);
        var o = t.vDRBih[n];
        return void 0 === o ? (void 0 === t.EOELbZ && (t.EOELbZ = !0),
            e = t.jRRxCS(e, r),
            t.vDRBih[n] = e) : e = o,
            e
    }
    return c(arg1, arg2)
}
function encode2(arg1, arg2) {
    var c = ["kmkRjCkHyG==", "tSkzhCooda==", "W5HyfwldN8oaq8kZWRj+fCkwCColW6pdVG==", "oNjak8o1", "W7ijFCk/zq==", "WQeJn8kMW54=", "W5TZqxn7W4NcJSo1WR4=", "WQfrW7JcOSocW5vs", "W74jevDO", "WO3dQSkcgJu=", "hKrxomoO", "jhBcNIrJ", "Emo/W53dGq==", "rMaLc3i=", "hmkKWPXWWQddJmkmWQC3", "W75cASo9WRKndmkl", "vConW4uZjq==", "gmkOnSkozG==", "EmkgWP/cMCkJWOib", "W6uKbffk", "wCkyWRhcR8km", "nNFcRYC=", "rv0Qd0C3FNlcGSk+WQy=", "WQdcObtdVSoVg8oHWPddNW==", "W4yRqSkPqq==", "WPGeb8kHW50=", "mcdcOmomW5xdLGBdQ2lcVeJdMmkWhmkD", "eSkQnSkz", "WPquomo0sq==", "wtVcRmkpW6m=", "A8klWPxcL8kd", "WP1qWP95WO0=", "WRNdQ2zLW7K=", "W4CcWOjBWRHvCG==", "WR1iW63cOCoBW5LnW7zVxh9r", "wLpdO8kqW4JcG8oG", "rCoGW7pdJmoW", "f8kHmCkkEuq=", "cmoJdmoUW7q=", "W5XDW6q=", "WQpdRKvKW7TRW6eYW7e=", "WPFdK8k9cdNcQKeSsa==", "WRLKW7/cHmoL", "w1mHpNi=", "DhyQhuq=", "W53dIrP1qa==", "W44Zz8k/", "W6BdPszHCG==", "WQz3W4/cPCoV", "CSkOWQngECkPWRNcPmkCW6ZcGCk3W6y=", "W5v+wmokWR8=", "xNqggwy=", "qCorzgxdQCoeW5ZcM1W=", "jmkYWObWWQe=", "jCovWQq0W5pcVa==", "tCoyW6pdKv0=", "xv4N", "nHO9WOyQW6G=", "aCk1WP1aWPC=", "W4uVjffacG==", "wSoGW5BdGMa=", "rCkShCoJ", "W5nMr8ojWQ4=", "uSk8WOFcQSkK", "W4TaW7ldUcW1l8kMWQZcL8ouW5S=", "WQ7cQe/dMCoWtbb5qSk3zeKbW5JcS8kL", "W6ldGZvkvSk3fx7cJG==", "lLb2lCoroGG=", "W7CJWOvkWOy=", "lfxcNSkJ", "s8k6WOhcU8kC", "W6VcKmo2hry=", "ymozW7q7Aa==", "CIX7rdK=", "W44RqCk5W5C=", "W558rN1t", "lHBcOmorW50=", "q8oZW5Kf", "BaNcUSkzW6v9AcRdKdWe", "W4HrW6xdGYK0hSkAWQG=", "D1WrcfK=", "W5VdRIrhWQtdG2K=", "W618C3XL", "W5eRjv1xpmoVWQ3dMq==", "mwtdISoNW6XgoCoVsa==", "W71Yx1PY", "W7uLv8k4W5q=", "W71QFurt", "WORcH3JdUmoj", "WRldO3r8W7u=", "pf3cJbfW", "FCodW5xdT1W=", "FmoFy2VdLq==", "WRJdRfLVW7TIW7aRW6qdW5O=", "WQG/nG==", "yCoJW5VdGCohW5qDA8oW", "bCoGWQCSwG==", "CCoWW7pdPsKhW4ZdG1ZcP8kjuvrd", "W5VdSd5uWQldMwpdV8oM", "emoNgmoiW5m=", "amkKWPf8WPS=", "W6OWzSkNEW==", "WRKTmmkYW50=", "W7SmwSkqW6q=", "F8oFzMhdQCod", "j1xcTmkGgq==", "W6RdNZzBsW==", "W4SVp3vao8o+WRZdGW==", "W4C3W7JcMdK=", "D8oMW6S7qa==", "y8olDgxdQCo9W5ZcHvRcRa==", "W4qEke5i", "gCkRWPTJ", "WOOogmk7W4NdIG==", "WRJdICkUhtNcVa==", "ySoFDMNdVmolW4hcHa==", "WP7cGfZdMCoe", "wvuPdLGMwMNcLW==", "W5vnp1tdSW==", "bLzAeCoK", "WRFdK8k9cdNcIKeSsmkjWP3dIWhdNmoNx8oeWQW=", "WRuKdSkmW4O=", "xSkHWQxcMmkc", "BqZdSmopW64=", "W7uoACk+W7jbW6ijWPu=", "mxFdHSo4W40=", "W5ailLzq", "d2ZcR8kalG==", "W7ddRtnkWQJdJM7cR8oqALldNcxdSb8xlmoTW5efDCkdW68kW7NcVgtdKmkhrGWTWPq=", "fmk1WRfvWQ8=", "nJOjWQqu", "DqpcT8kY", "WQrbWP1hWOu=", "W7hdPGTsWOa=", "xv0Nagu=", "WO7dK8k9gdtcVvO6vmk4", "evxdV8ocW48=", "bmoWWPabW7W=", "W7LaW77dJsT4gSkuWQ3cMG==", "W5vxW4hdJY4=", "u8oQW483hG==", "W7a5nw1s", "W51AhNFdHmorACkMWQu=", "cmkXpCkEEv7dLSo6pq==", "WQBcVHZdSSo9", "WOSueSk/W43dIG==", "qCosW67dPmoK", "W5GwWPrJWRrwCfHj", "W7/dNIvTwSk+h1RcLfGvCq==", "W4RdNJjwqq==", "sui0oM8=", "y8kkWQriCq==", "W7z2W43dJXe=", "vcFdHSo6W5S=", "dLbMkmotkYiCg8o8yCojW61FWQhcKYC1WPJcMSoxBq==", "jmotWRa+W43cOSkJaW==", "W5uTnvzjoConWQFdMW==", "WPiGkmozzCodDmoRva==", "AGddJmoPW4S=", "W4qqASk2ta==", "FxSNcgO=", "B8osAwxdTCoEW60=", "WRzjW7tcJ8oBW45kW6H6swrkW7m=", "WQlcQvJdR8oNtHTDB8k9Fa==", "WPO0oCkRW6u=", "lvRcMCkZf29ZW5O2WQBcUq==", "W5qUW7tcKdRcGmkCs8oZ", "WOSXgCkVW4u=", "W4SHmKPaomo2WR7dJG==", "FGZcVCkT", "qh0VkKqwmxRcIW==", "bmo7WPu+W44=", "W69sogldKq==", "WPSGjmo0", "awJcJSk8pG==", "zmkhpmoojG==", "W53dOqnCqG==", "xG7cQCkIW4C=", "x8k5WO/cL8ki", "umohW6hdHSo9", "W6VcK8o2", "etWLWQGJ", "W5/dRsrdWQxdNM7dRSoXFW==", "nxdcTdv1", "W5eHW7pcNHi=", "xIJcTSkqW4K=", "WQxcRXpdSmoh", "BqxcImkbW6q=", "WQmGj8kWW5tdOgeFWR5gW5BdNa==", "WQFdQfvVW6vUW4m4W7m=", "hmkOlCkSra==", "s8kHAcSz", "iSo1WOeABmoLW705", "WQBcRqldVSoSha==", "xCo6W7BdG8oT", "DCklWPJcK8ksWPu3W47dKCklW4DWW4Ty", "vh0TifW=", "CXJcQSkJW6jgAdhdQd0u", "jrmSWOij", "WO7cRw3dPCod", "WQf1W6RcOmoh", "WQVcHwhdTmoC", "gmkOoSkmF2/dNSo3mHO=", "WPOrgSkXW5W=", "W5qbWO1gWR1VFKHvfG==", "rCo9W5KBzSkoWR3cOvuGW4CUW5TCgq==", "v8oRW5ZdN8oh", "fCoKWOCFBSo0W5CIW5NcI8kI", "W6RcT8owpqK=", "p8oyWR8V", "W4DBbhNdMq==", "q8kLWPbMBG==", "beZcTdzw", "b2KYtea=", "uSktWQ/cNCkz", "tmkKWQBcLSk+", "nSojiSoFW6BcSsa+W4C=", "W7SMzCkOW68=", "BmocW4K9CG==", "m3SYrMi=", "i3/dI8o3", "WQxcVb/dR8oMbSo2WOxdNG==", "z8oEW6elkG==", "W47dSsDcWRu=", "W5TUggZdNG==", "pe4VsW==", "lLP9amofoGide8oTzSosW6jOWQFcKJ0cWOhcK8ovFmkK", "W4qNFSk8W4eV", "kcVcOmoxW53dLXC=", "W5aAWOvB", "WObbWRjYWRm=", "qCkmWOXaAa==", "WRRdOL5L", "seOHbv8=", "mCozWQu=", "WQvoW4KqW4u=", "WP8ieSkRW7q=", "W55yhwRdNW==", "zKeYega=", "w2xdOmksW4a=", "W5WzWOvB", "W7OBrmk6W7O=", "eSoWWP0ECmozW7C9W5VcJCkI", "u8kgWRbJtG==", "vZH7AcG=", "auaS", "h8oRWQOmya==", "W63cT8o8gs0=", "WOiClCksW7m=", "vmktWQn9vW==", "omoxWOCkyW==", "W7r6gvhdJW==", "W5SfW4hcTY0=", "W7yMFCk5zNi=", "fmkQWPfIWRJdImkfWRy=", "wLFdVCkyW4BcJq==", "WQBcOKldQa==", "b3NcMYPe", "wSkpwGmD", "WPjMWQ98", "cmkmhCkFqa==", "WPzhW63cQW==", "mNFcQdbPv8oOF1y=", "WQf+W7WqW4O=", "tSkTemoU", "WRPuW7ZcQa==", "yCoZW5C=", "uCo6W7xdT2WLW4xdK2O=", "W4n8xvP4W47cH8oKWRi=", "tmocW48S", "aulcNCkufa==", "feeT", "W4hcLCopbbu=", "W6VdPqPrAq==", "rSoaW487amolp2FcHCkejmkkucW=", "W5ONwmkUW70=", "e2D4e8ou", "xhOhihO=", "W7dcU8o2gZ0=", "WPZcGw7dKmov", "W5TTqxDPW4xcS8o1WQJdTuNdH8oXWOvNW6m=", "h8kLk8km", "W5VdTYjiWOpdGM7dPSoLyLFdNcpdSciC", "WQKUmSkSW57dPhSeWOe=", "WO3cIsBdTCoe", "W7yfESkYFa==", "smk+AsG/", "W6mfW7JcOWu=", "uYnUwsm=", "CmkGWPxcKCkO", "keZdGCohW6e=", "W6JcPmoAbru=", "ofb+jCovpaGC", "W71VeMddQG==", "WPNdM0zDW74=", "WPflW47cHCok", "W7LtDxXU", "W7ehW7pcLH0=", "W79Pu2bw", "efK6sLNdTrfJWRZdPum=", "gNGFr34=", "W5DPySo9WO8=", "WO8LnmokDSojya==", "k8kwg8kIEa==", "sLKWlKC3vMhcICkKWPddVwuY", "WOpcP2NdQSod", "qvJdUSki", "W6WHWPzRWRu=", "nmo8WRaAvG==", "W4uIwSkjwG==", "j2tdISo+W4bAiCoTBHC1lq==", "ba/cTmoUW4e=", "W4qMzCk0AMxdR8opu1LXEdlcGSokgSkV", "tmkch8o+iG==", "nhJdGCo2W6vBlSo6sq==", "iSkcWQvLWRm=", "tmo0W6pdR0C=", "W73dJcnUWOy=", "qI5Fqs04uCkyW44=", "tSoDW6OgCG==", "WOODq8kmWOS=", "W4JdQInpWQddIa==", "qwOXj14=", "nmoyWPuSW50=", "umoFW4mQkSoPlgZcNW==", "WOxcJ2JdImoh", "WPyinSonqq==", "W73cOCo6pI4=", "D8obW5VdVCoE", "WR/dRSkMcJ0=", "cSo0aSo2W7dcQsq+W5ldVfO=", "W4ThW6tdHa==", "mrZcH8o4W5G=", "WOzMWRH2WOG=", "W5SjF8k0W61k", "CJddLSo+W6DgESk0gmkK", "W7/cRvO=", "ACoqy2/dV8op", "DSo9W4BdTmoH", "AdVdJCo8", "W7uHpxvk", "WPxdICk8hI7cMuC/uSkK", "W5/dPYju", "b1LGi8oi", "nCkDWPr5WOq=", "cSkqWRDcWOm=", "uSovW7hdOCoG", "WPWkg8ktW78=", "W4ObW7BcKra=", "WPnnW5aSW5DrWRO=", "W6VcG8o6aJDYWOL+CG==", "qCovW7q/ga==", "msRcSmoEW4ddMaZdLuRcSuxdPa==", "nHmJWOuxW6u3CCkoWPpdPW==", "s1NdVmkxW4dcHq==", "W6iQW5pcNtm=", "W4KAvCktW7C=", "qg4Jnwu=", "bee/rLpdLbPVWR8=", "aSkUWRHEWQy=", "WQddUhX7W44=", "W4vbaNFdHmoxAq==", "s1a3ceW=", "pINcUSoCW58=", "WOiJemksW6m=", "ir06WOOVW54IFSkiWOJdJXhcNCoLFSo3W7yrW6W=", "qCoUC1pdOG==", "W4tdJqfiWRq=", "WOpdUM9zW5K=", "nLdcSJLc", "WPDhW5dcMSo9", "W4mrWPz1WR8=", "WPbxWRrvWRa=", "W5XyhLtdQq==", "W7mMwSkkW4y=", "ltFcTSoRW53dNaBdQhFcVK7dUW==", "W4Heq8ovWPG=", "gCoKWP0A", "m3pcSbHw", "WQFdQfv4W6nOW4C4", "W6zbsSoTWOK=", "s17dSSksW47cHCoHqXWin1yTDG==", "qg4Ylu4RjN4=", "WPqKkCoM", "l3BcTcC=", "wCkjWOhcMmkA", "W7DPBej/", "WOixiSkRW6G=", "W7ycavnq", "WOzpWRr3WOu=", "W64wF8kpW7C=", "WQfjW7tcQW==", "WQeGnSkaW5JdPMC=", "W6HLW67dHde=", "kCozgCoFW4i=", "WRRcOK/dUCoGqbbOAG==", "W4eGzmkqW7C=", "zZZdImo8W6Dg", "WOxcM3pdI8ot", "W5uIlLPa", "W7PQv3fP", "nSkulmk+Da==", "WQhcO1W=", "WQjhW7RcPCoG", "W6WOE8k0W4S=", "gMvNbSoH", "WQW2eSkGW44=", "xCkOrGyi", "W4KZF8kY", "WQScaCk8W78=", "W4WoEmk4W6HcW6qfWOi=", "xLmPdG==", "W6BdGILn", "W6y6WQLJWOi=", "WRVcQYBdUmoI", "W4ldPaboWQm=", "A8kCtbaK", "zCoCW5aVBW==", "bGy2WOuIW4aZE8ktWP0=", "fmoWWQWsW6W=", "y1G5nL8=", "ighcUcrI", "cmkLoCkmF0u=", "cCoPWQOkrG==", "yCkHWQLbuW==", "WOtcPZtdL8o5", "mH08", "WRTNW7GdW6G=", "ifFcKSk6hMrcW6u3", "smkZhmoOdW==", "qs9o", "gmojbCoZW6a=", "jxFdKCoY", "WRPKWPfnWPi=", "EmkUWQ5pzCk5WQ8=", "W50zFCk0W7jBW7G=", "W5ZdLbTbWQq=", "WQ8jj8kSW6a=", "WQfZW6OCW616WPS=", "mNFcJIDZu8oPBG==", "W6y6DSkQAG==", "zCkfa8otpq==", "WOZcHbFdISo8", "F8oWW5RdMSo3W5mqDmoNW7mrttWsFq==", "lmoJWPmoW6K=", "eSoUWOGsoSkxW6pcQsq=", "vheWd28=", "WPi8WQlcIwJcLCoduSkIW4NcMW==", "W5P8v3f4W5q=", "b8o2pCoZW4y=", "W4DZtgi=", "i0ZcN8k6hG==", "WRhcVJpdMCoZ", "lCkWdSk4rG==", "W7NdIJPJxq==", "WQD5W6uHW6O=", "i8ogWRi6W4VcTCkvfdv3W4CqiCoNWRtdPa==", "c8kLpmkgqW==", "ECkCrdG/WQH8", "smo8W5mA", "W4PAW4hdQZe=", "W5VdOZjlWOm=", "hSkKWOz+WQpdImolWQeRWPtdPa==", "cfFcH8k1aW==", "EmkAWQ5+FW==", "A8kTWQBcLSki", "WPNdLmk6fdhcQW==", "l8obn8o2W5dcQYyNW58=", "sCkGwIii", "sGVcL8kwW74=", "CmoEW4qQmG==", "W488zq==", "WOarfCkkW43dKgRdHSoGsKK=", "lhFdLq==", "kCktWOHtWRe=", "rv0TguC7vwe=", "nx/dImo2W5bgiCoYxq==", "W4f3W4BdRJq=", "WRRcP0BdL8or", "n1ddJmo8W7y=", "WQnRW7RcM8o6", "W4pcTSodgbu=", "sCoZW5qkz8koWPBcO3uIW5y=", "v8kXfSoUaqDtgSoW", "WRGimSkuW5G=", "pSoxWQuuW4JcVSkwaYHXW4CqaCo3", "hfnzeCoE"];
    n = c,
        e = 458,
        function (t) {
            for (; --t;)
                n.push(n.shift())
        }(++e);
    var W = function t(n, r) {
        var e = c[n -= 0];
        void 0 === t.GMJOxm && (t.CPxjpy = function (t, n) {
            for (var r = [], e = 0, o = void 0, a = "", i = "", u = 0, c = (t = function (t) {
                for (var n, r, e = String(t).replace(/=+$/, ""), o = "", a = 0, i = 0; r = e.charAt(i++); ~r && (n = a % 4 ? 64 * n + r : r,
                    a++ % 4) ? o += String.fromCharCode(255 & n >> (-2 * a & 6)) : 0)
                    r = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/=".indexOf(r);
                return o
            }(t)).length; u < c; u++)
                i += "%" + ("00" + t.charCodeAt(u).toString(16)).slice(-2);
            t = decodeURIComponent(i);
            var W = void 0;
            for (W = 0; W < 256; W++)
                r[W] = W;
            for (W = 0; W < 256; W++)
                e = (e + r[W] + n.charCodeAt(W % n.length)) % 256,
                    o = r[W],
                    r[W] = r[e],
                    r[e] = o;
            W = 0,
                e = 0;
            for (var d = 0; d < t.length; d++)
                e = (e + r[W = (W + 1) % 256]) % 256,
                    o = r[W],
                    r[W] = r[e],
                    r[e] = o,
                    a += String.fromCharCode(t.charCodeAt(d) ^ r[(r[W] + r[e]) % 256]);
            return a
        }
            ,
            t.hpBrye = {},
            t.GMJOxm = !0);
        var o = t.hpBrye[n];
        return void 0 === o ? (void 0 === t.HWFFId && (t.HWFFId = !0),
            e = t.CPxjpy(e, r),
            t.hpBrye[n] = e) : e = o,
            e
    }
    return W(arg1, arg2)
}

function find_path_reference(scope, txt) {
    if (!scope.referencePaths) return;
    for (let i = 0; i < scope.referencePaths.length; i++) {
        const ref = scope.referencePaths[i];
        const varPath = ref.parentPath;
        const propPath = varPath.get("property")
        if (varPath.key == "left" && t.isLiteral(propPath.node) && propPath.node.value == txt) {
            // if(txt=="YIUXn")debugger
            return varPath.parentPath.get("right")
        }
    }
}

//还原字符串
traverse(ast, {
    CallExpression(path) {
        const node = path.node;
        if (node.arguments.length == 2 && node.arguments[0].type == "StringLiteral" && node.arguments[1].type == "StringLiteral") {
            try {
                //解密
                const val = encode2(...path.node.arguments.map(x => x.value))
                if (!val) {
                    console.log("跳过")
                    return;
                }
                // console.log(path.toString(), val)
                path.replaceWith(t.stringLiteral(val))
                //结束当前函数递归
                // path.skip();
            } catch (e) {
                console.log(e)
            }
        }
    }
})

//替换表达式
function rpc_reg_fn(basePath, rpsPath) {
    const yargs = basePath.node.arguments;
    if(!yargs || yargs.filter(x=>t.isFunction(x)).length){
        return;
    }
    //值是方法
    if (rpsPath.node.type == "FunctionExpression") {
        //参数名
        const args = {};
        rpsPath.get("params").forEach((x, i) => {
            args[x.node.name] = yargs[i]
        })
        //获取表达式
        const temp_node = t.cloneNode(rpsPath.get("body.body.0.argument").node, true);
        if (!temp_node) {
            return;
        }
        traverse(temp_node, {
            Identifier(path) {
                const rg = args[path.node.name];
                if (!rg) {
                    return;
                }
                try {
                    path.replaceWith(t.cloneNode(rg, true))
                    path.skip();
                } catch (e) {
                    console.log(basePath,rpsPath)
                    debugger
                }
            }
        }, {
            noScope: true
        })
        try {
            basePath.replaceWith(temp_node);
        } catch (e) {
            // console.log(e, basePath.toString())
        }
    } else if (t.isLiteral(rpsPath.node)) {
        //固定内容
        try {
            basePath.replaceWith(t.stringLiteral(rpsPath.node.value))
        } catch (e) {
            // debugger
        }
    }

}

/*

//还原函数basePath
traverse(ast, {
    MemberExpression(path) {
        let prop = path.get("property");
        if (prop.type != "StringLiteral" || path.key == "left") {
            //不符合格式
            return;
        }
        let rpPath = path;
        const basePath = path.parentPath;
        //查找变量
        function deep(path, keyPath) {
            const objBind = get_scope_binding(path);
            if (!objBind) return;
            const initPath = objBind.path.get("init")
            if (objBind.kind == "var" && initPath.type == "Identifier") {
                //赋值
                return deep(initPath, keyPath)
            }
            //返回
            const rpsPath = find_path_reference(objBind, keyPath.node.value);
            if (!rpsPath) {
                return;
            }
            if (basePath.node.type == "CallExpression") {
                rpPath = basePath;
            }
            rpc_reg_fn(rpPath, rpsPath);
            //参数对应
            // keyPath.replaceWith(rpsPath.node.__clone())

            // console.log(rpsPath)
        }
        deep(path, path.get("property"))
        // path.skip();
    }
})



//恢复字符串
traverse(ast, {
    Identifier(path) {
        if (!(["init", "property"].includes(path.key))) {
            return;
        }
        if (path.node.name == "k") {
            // debugger
        }
        //执行
        const evl = path.evaluate();
        if (!evl || !evl.value) {
            return;
        }
        path.replaceWith(t.valueToNode(evl.value));
        path.skip();
    }
})
*/


/* 
// 删除无用r['xxx']=
traverse(ast, {
    MemberExpression(path) {
        if (path.get("property").type != "StringLiteral" || path.parentPath.type != "AssignmentExpression") {
            return;
        }
        // const key=path.get("property").node.value;
        const binding = get_scope_binding(path);
        if (!binding) {
            return;
        }
        //获取
        const ls = binding.referencePaths.filter(x => x.parentPath.key != "left")
        if (ls.length < 2) {
            path.parentPath.remove();
        }

    }
}) 

//删除没用的var e = r

//转源码，必须转一下否则数据不刷新
let deobfCode = generator(ast, { comments: false }).code;
//在转ast
ast = parse(deobfCode)
traverse(ast, {
    "VariableDeclarator|FunctionDeclaration"(path) {
        const { node, scope } = path;
        const bind = scope.getBinding(node.id.name);
        if (!bind) return;
        const { constant, referenced, referencePaths } = bind;
        // If the variable is constant and never referenced, remove it.
        // if(path.toString()=="e = {}")debugger
        console.log(path.toString(), referencePaths.length)
        if (constant && !referenced) {
            path.remove();
            path.skip()
        }
    }
})





// 数组转点优化
traverse(ast, {
    MemberExpression(path) {
        const prop = path.get("property");
        if (prop.node.type == "StringLiteral") {
            path.replaceWith(t.memberExpression(t.cloneNode(path.node.object, true), t.identifier(prop.node.value)))
        }
    },

    // StringLiteral(path) {
    //     if (!(path.key == "property" && /^[a-zA-Z\d]+$/.test(path.node.value))) {
    //         return;
    //     }
    //     path.parentPath.replaceWith(t.memberExpression(t.cloneNode(path.parent.object, true), t.identifier(path.node.value)))
    // }
})

traverse(ast, {
    EmptyStatement(path) {
        path.remove();
    },
})

*/

//美化代码
deobfCode = generator(ast, { comments: false }).code;
deobfCode = beautify(deobfCode, {
    indent_size: 2,
    space_in_empty_paren: true,
});


fs.writeFileSync("pdd_encode.js", deobfCode)
```

### 修复

以下是使用 nodejs 的修复版本，放到文件最上面

```
const window = Object.assign(global, {
    outerHeight: 100,
    outerWidth: 100,
    location: {
        href: "https://www.temu.com",
        port:""
    },
    document: {
        referrer: "",
        cookie: ""
    },
    screen: {
        availWidth: 2560,
        availHeight: 1400
    },
    navigator: {
        userAgent:"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36 Edg/110.0.1587.57"
    }
});
```

### 核心算法

```
function (t, n, r) {
    "use strict";
    (function init(t) {
        var a = r(5);
        // var a = {};
        var i = r(3);
        // var i = {};
        //--------
        var U = 0,
            $ = [],
            E = void 0,
            Y = void 0,
            X = 0,
            tt = function () { },
            nt = window;
        var rt = window.navigator;
        //"ontouchstart" in nt.document 移动端还是PC端
        var it = false;
        //("undefined" == typeof process ? "undefined" : typeof process) === "undefined" ? null : process; //浏览器环境直接为null
        var ut = null;

        var dt = function () {
            var a = [];
            // if (typeof window.outerHeight !== "number" || typeof window.outerWidth !== "number") {
            //     a[0] = 1
            // } else {
            //     a[0] = window.outerHeight < 1 || window.outerWidth < 1 ? 1 : 0
            // }
            a[0] = 0;
            //typeof nt.callPhantom !== "undefined" || typeof nt._phantom !== "undefined" ? 1 : 0;
            a[1] = 0;
            //typeof nt.Buffer === "undefined" ? 0 : 1;
            a[2] = 0;
            //typeof nt.emit === "undefined" ? 0 : 1;
            a[3] = 0;
            //typeof nt.spawn === "undefined" ? 0 : 1;
            a[4] = 0;
            //rt.webdriver === !0 ? 1 : 0;
            a[5] = 0;
            //typeof nt.domAutomation === "undefined" && typeof nt.domAutomationController === "undefined" ? 0 : 1;
            a[6] = 0;

            // try {
            //     typeof Function.prototype.bind == "undefined" && (a[7] = 1);
            //     Function.prototype.bind.toString().replace(/bind/g, "Error") !== Error.toString() && (a[7] = 1);
            //     Function.prototype.toString.toString().replace(/toString/g, "Error") !== Error.toString() && (a[7] = 1);
            // } catch (t) {
            //     a[7] = 0;
            // }
            a[7] = 0;
            //rt.plugins && rt.plugins.length === 0 ? 1 : 0;
            a[8] = 0;
            // rt.languages === "" ? 1 : 0
            a[9] = 0;
            //nt.vendor === "Brian Paul" && nt.renderer === "Mesa OffScreen" ? 1 : 0
            a[10] = 0;
            //nt.Modernizr && !nt.Modernizr.hairline ? 1 : 0
            a[11] = 0;
            //nt.chrome === void 0 ? 1 : 0;
            a[12] = 0;
            //"webdriver" in rt ? 1 : 0;
            a[13] = 0;
            //  window.navigator.hasOwnProperty("webdriver") ? 1 : 0;
            a[14] = 0;
            //window.history.back && window.history.back.toString().indexOf("ipcRenderer") > -1 ? 1 : 0;
            a[15] = 0;
            //child_process
            a[16] = 0;
            //nt.document.getElementById.toString().indexOf("native code") > -1 ? 0 : 1;
            a[17] = 0;
            return a;
        };

        function xt(t, n, r) {
            var i = {},
                u = n || nt.event;
            if (u.timeStamp > 0) {
                if (t.preTimeStamp && u.timeStamp - t.preTimeStamp < 15) {
                    return;
                }
                t.preTimeStamp = u.timeStamp;
            }
            var c = {};
            c.elementId = u.target.id || "";
            c.timestamp = Date.now() - U;
            var W = u.changedTouches;
            if (W && W.length) {
                c.clientX = W[0].clientX, c.clientY = W[0].clientY
            } else {
                c.clientX = u.clientX, c.clientY = u.clientY
            }
            if ((void 0 === r ? "undefined" : typeof r) !== "undefined") {
                t.data[r].push(c);
                if (t.data[r].length > t.maxLength) {
                    t.data[r].shift()
                }
            } else {
                t.data.push(c);
                if (t.data.length > t.maxLength) {
                    t.data.shift()
                }
            }

        }
        //cookie中获取
        function ft(t) {
            var o = {};
            return (nt.document.cookie ? nt.document.cookie.split("; ") : []).some(function (r) {
                var i = r.split("="),
                    u = i.slice(1).join("="),
                    c = i[0].replace(/(%[0-9A-Z]{2})+/g, decodeURIComponent);
                return u = u.replace(/(%[0-9A-Z]{2})+/g, decodeURIComponent), o[c] = u, t === c
            }), t ? o[t] || "" : o;
        }

        //events
        function st(t) {
            if (!t || !t.length) {
                return []
            };
            var n = [];
            t.forEach(function (t) {
                var r = i.sc(t.elementId);
                n = n.concat(i.va(t.clientX),
                    i.va(t.clientY),
                    i.va(t.timestamp),
                    i.va(r.length),
                    r
                );
            })
            return n;
        }

        var lt = {};
        lt.data = [];
        lt.maxLength = 1;
        lt.init = function () {
            var o = i.gos(vt, it ? "touchStartEventData" : "MouseDownEventData");
            this.c = i.pbc("clickEventData" + o);
        };
        lt.handleEvent = function (t) {
            xt(this, t)
        };
        lt.packN = function () {
            if (this.data.length === 0)
                return [];
            var e = [].concat(i.ek(4, this.data), st(this.data));
            return e.concat(this.c);
        };

        var vt = {};
        vt.data = [];
        vt.maxLength = 1;
        vt.handleEvent = function (t) {
            xt(this, t)
        };
        vt.packN = function () {
            if (this.data.length === 0)
                return [];
            return [].concat(i.ek(it ? 1 : 2, this.data), st(this.data));
        };
        var _t = {};
        _t.data = [];
        _t.maxLength = 30;
        _t.handleEvent = function (t) {
            if (it) {
                if (!this.data[X]) {
                    this.data[X] = []
                }
                xt(this, t, X)
            } else {
                xt(this, t);
            }
        };
        _t.packN = function () {
            var e = [];
            if (it) {
                e = this.data.filter(function (t) {
                    return t && t.length > 0;
                });
                for (var o = 0, a = e.length - 1; a >= 0; a--) {
                    o += e[a].length;
                    var u = o - this.maxLength;
                    if (u > 0 && (e[a] = e[a].slice(u)), u >= 0) {
                        e = e.slice(a);
                        break;
                    }
                }
            } else {
                e = this.data
            };
            if (e.length === 0) {
                return [];
            }
            var c = [].concat(i.ek(it ? 24 : 25, e));
            if (it) {
                e.forEach(function (n) {
                    c = (c = c.concat(i.va(n.length))).concat(st(n));
                })
            } else {
                c = c.concat(st(this.data))
            }
            return c;
        };

        //浏览器滚动条高度 documentElement.scrollTop document.body.scrollTop
        var pt = {};
        pt.data = [];
        pt.maxLength = 3;
        pt.handleEvent = function () {
            var e = {},
                o = nt.document.documentElement.scrollTop || nt.document.body.scrollTop;
            if (o > 0) {
                e.scrollTop = o;
                e.timestamp = Date.now() - U;
                this.data.push(e);
                if (this.data.length > this.maxLength) {
                    this.data.shift()
                }
            }
        };
        pt.packN = function () {
            if (it && this.handleEvent(), !this.data.length) {
                return [];
            }
            var t = [].concat(i.ek(3, this.data));
            this.data.forEach(function (n) {
                t = t.concat(i.va(n.scrollTop), i.va(n.timestamp));
            })
            return t;
        };
        //路由方面 location.href location.port
        var gt = {};
        gt.init = function () {
            this.data = {};
            this.data.href = nt.location.href;
            this.data.port = nt.location.port;
            this.c = i.pbc("locationInfo");
        };
        gt.packN = function () {
            var e = i.ek(7);
            var u = void 0 === this.data.href ? "" : this.data.href;
            var W = void 0 === this.data.port ? "" : this.data.port;
            if (!u && !W) {
                return [].concat(e, this.c);
            }
            var x = u.length > 128 ? u.slice(0, 128) : u;
            var f = i.sc(x);
            return [].concat(e, i.va(f.length), f, i.va(W.length), W.length === 0 ? [] : i.sc(this.data.port), this.c);
        };
        //设备宽度和高度 screen.availWidth screen.availHeigh
        var wt = {};
        wt.init = function () {
            this.data = {};
            this.data.availWidth = nt.screen.availWidth;
            this.data.availHeight = nt.screen.availHeight;
        };
        wt.packN = function () {
            return [].concat(i.ek(8), i.va(this.data.availWidth), i.va(this.data.availHeight));
        };
        //随机数
        var Rt = {};
        Rt.init = function () {
            this.data = nt.parseInt(Math.random() * (Math.pow(2, 52) + 1).toString(), 10)
                + nt.parseInt(Math.random() * (Math.pow(2, 30) + 1).toString(), 10)
                + "-" + E;
        };
        Rt.packN = function () {
            return this.init(), [].concat(i.ek(9, this.data));
        };
        //document
        var Ot = {};
        Ot.data = [];
        Ot.init = function () {
            var n = {};
            n.CjCho = function (t) {
                return t();
            };
            this.data = dt();
        };
        Ot.packN = function () {
            try {
                this.data[18] = Object.keys(nt.document)
                    .some(function (n) {
                        return nt.document[n] && nt.document[n].cache_;
                    }) ? 1 : 0;
            } catch (t) {
                this.data[18] = 0;
            }
            for (var e = 0, o = 0; o < this.data.length; o++) {
                e += this.data[o] << o;
            }
            return [].concat(i.ek(10), i.va(e));
        };
        //location.href
        var Pt = {};
        Pt.init = function () {
            this.data = i.pbc(nt.location.href ? nt.location.href : "");
        };
        Pt.packN = function () {
            return this.data.toString().length ? [].concat(i.ek(11), this.data) : [];
        };
        // window.DeviceOrientationEvent
        var zt = {};
        zt.init = function () {
            this.data = nt.DeviceOrientationEvent ? "y" : "n";
        };
        zt.packN = function () {
            return [].concat(i.ek(12, this.data));
        };
        //window.DeviceMotionEvent
        var Jt = {};
        Jt.init = function () {
            this.data = nt.DeviceMotionEvent ? "y" : "n";
        };
        Jt.packN = function () {
            return [].concat(i.ek(13, this.data));
        };
        //当前时间减去服务器更新时间
        var Bt = {};
        Bt.init = function () {
            this.data = Date.now() - Y;
        };
        Bt.packN = function () {
            return this.init(), [].concat(i.ek(14, this.data));
        };
        //userAgent
        var At = {};
        At.init = function () {
            this.data = rt.userAgent;
        };
        At.packN = function () {
            return this.data.length ? [].concat(i.ek(15, this.data)) : [];
        };
        var u = r(14)
        var Vt = {};
        Vt.init = function () {
            this.data = u();
        };
        Vt.packN = function () {
            var t = this;
            var o = [],
                a = {};
            a.nano_cookie_fp = 16;
            a.nano_storage_fp = 17;
            Object.keys(this.data).forEach(function (n) {
                var r = [].concat(t.data[n] ? i.ek(a[n], t.data[n]) : []);
                o.push(r);
            })
            return o;
        };
        //document.referrer
        var jt = {};
        jt.init = function () {
            var e = nt.document.referrer || "",
                o = e.indexOf("?");
            this.data = e.slice(0, o > -1 ? o : e.length);
        };
        jt.packN = function () {
            return this.data.length ? [].concat(i.ek(18, this.data)) : [];
        };
        //pdd_user_id
        var Nt = {};
        Nt.init = function () {
            this.data = ft("pdd_user_id");
        };
        Nt.packN = function () {
            return this.data.length ? [].concat(i.ek(19, this.data)) : [];
        };
        //api_uid
        var Tt = {};
        Tt.init = function () {
            this.data = ft("api_uid");
        };
        Tt.packN = function () {
            return this.data.length ? [].concat(i.ek(20, this.data)) : [];
        };
        //
        var Ut = {};
        Ut.data = 0
        Ut.packN = function () {
            return [].concat(i.ek(21, this.data));
        };
        //
        var Yt = {};
        Yt.init = function (t) {
            this.data = t;
        };
        Yt.packN = function () {
            return [].concat(i.ek(22, this.data));
        };
        //pdd_vds
        var $t = {};
        $t.init = function () {
            this.data = ft("pdd_vds");
        };
        $t.packN = function () {
            return this.data.length ? [].concat(i.ek(23, this.data)) : [];
        };

        //检测浏览器
        var nn = {};
        nn.init = function () {
            var e = [
                nt.opr || nt.opera || rt.userAgent && " OPR/" > -1 ? 1 : 0,
                ("undefined" == typeof InstallTrigger ? "undefined" : typeof InstallTrigger) !== "undefined" ? 1 : 0,
                /constructor/i.test(nt.HTMLElement) || (nt.safari && nt.safari.pushNotification || "").toString() === "[object SafariRemoteNotification]" ? 1 : 0,
                nt.document && nt.document.documentMode || nt.StyleMedia || nt.navigate ? 1 : 0,
                nt.chrome && (nt.chrome.webstore || nt.chrome.runtime) ? 1 : 0
            ];
            for (var i = 0, a = 0; i < e.length; i++) {
                a += e[i] << i;
            }
            this.data = a;
        };
        nn.packN = function () {
            return [].concat(i.ek(26), i.va(this.data));
        };

        function rn(t) {//Tt nn lt
            [wt, Ot, Pt, zt, Jt, At, Vt, jt, Nt, Tt, Yt, $t, gt, nn, lt].forEach(function (n) {
                n.init(t);
            });
        }
        //鼠标按下和鼠标移动
        function en() {
            return;
            //添加移动日志
            var e = "mousedown",
                o = "mousemove";
            it && (e = "touchstart", o = "touchmove");
            nt.document.addEventListener(e, vt, !0);
            nt.document.addEventListener(o, _t, !0);
            nt.document.addEventListener("click", lt, !0);
            !it && nt.document.addEventListener("scroll", pt, !0);
        }

        function on() {
            X = 0;
            [vt, _t, lt, pt].forEach(function (t) {
                t.data = [];
            });
        }

        function an() {
            var e = i.pbc(dt.toString() + un.toString());
            $ = e.map(function (t) {
                return String.fromCharCode(t);
            });
        }


        function un() {
            if (!nt) return "";
            var t = [];
            var u = t.concat.apply(
                t,
                [
                    vt.packN(),
                    _t.packN(),
                    lt.packN(),
                    pt.packN(),
                    gt.packN(),
                    wt.packN(),
                    Rt.packN(),
                    Ot.packN(),
                    Pt.packN(),
                    zt.packN(),
                    Jt.packN(),
                    Bt.packN(),
                    At.packN()
                ].concat(
                    (function (t) {
                        if (Array.isArray(t)) {
                            for (var n = 0, r = Array(t.length); n < t.length; n++) r[n] = t[n];
                            return r;
                        }
                        return Array.from(t);
                    })(Vt.packN())
                    , [
                        jt.packN(),
                        Nt.packN(),
                        Tt.packN(),
                        Ut.packN(),
                        Yt.packN(),
                        $t.packN(),
                        nn.packN()
                    ]
                )
            );
            //延迟加载
            setTimeout(function () {
                on();
            }, 0);
            var c = u.length.toString(2).split("");
            for (var W = 0; c.length < 16; W += 1) {
                c.unshift("0");
            }
            c = c.join("");
            var x = [];
            if (u.length === 0) {
                x.push(0, 0)
            } else if (u.length > 0 && u.length <= (1 << 8) - 1) {
                x.push(0, u.length)
            } else if (u.length > (1 << 8) - 1) {
                x.push(
                    nt.parseInt(c.substring(0, 8), 2),
                    nt.parseInt(c.substring(8, 16), 2)
                );
            }
            u = [].concat([3], [1, 0, 0], x, u);
            var f = a.deflate(u);
            var s = [].map.call(f, function (t) {
                return String.fromCharCode(t);
            });
            return "0aq" + i.encode(s.join("") + $.join(""), i.budget);
        }

        function cn(t = {}) {
            var t = arguments.length > 0 && void 0 !== arguments[0] ? arguments[0] : {};
            if ((void 0 === nt ? "undefined" : typeof nt) !== "undefined") {
                //更新服务器时间
                this.updateServerTime(t.serverTime || 879609302220);
                //当前时间
                U = Date.now();
                //初始化
                rn(U, nt);
                //
                en();
                an();
            }
        }

        cn.prototype.updateServerTime = function (t) {
            Y = Date.now();
            E = t;
        };
        cn.prototype.init = tt;
        cn.prototype.clearCache = tt;
        cn.prototype.messagePack = function () {
            return Ut.data++, un();
        };
        cn.prototype.messagePackSync = function () {
            return new Promise(function (n) {
                Ut.data++, n(un());
            });
        };
        //这里如果是node环境就添加 swallow
        if (ut && ut.env && ut.env.BROWSER) {
            cn.prototype.swallow = function (t) {
                switch (t.type) {
                    case "click":
                        lt.handleEvent(t);
                        break;
                    case "touchstart":
                    case "mousedown":
                        vt.handleEvent(t);
                        break;
                    case "touchmove":
                    case "mousemove":
                        _t.handleEvent(t);
                }
            }
        }
        t.exports = cn;
        // return cn;
    }).call(this, r(1)(t))
}
```