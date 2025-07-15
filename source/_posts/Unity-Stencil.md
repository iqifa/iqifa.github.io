---
title: Unity_Stencil
date: 2024-10-29 00:31:05
categories: [Unity,渲染]
---
### Unity 模板测试

模板测试在Opengl里已经了解很多了，这里主要写Unity ShaderLab中的一些问题。简单实现下模板测试原理的轮廓效果，如下。

![轮廓](Image1.png)

其中一些问题

1. URP里多Pass，需要在Pass里面标记Tags才能正常渲染

	     Tags{"LightMode"="*******"}
具体枚举在URP包里的 Runtime/Passes 里的 DrawObjectsPass.cs脚本，有如下代码

        public DrawObjectsPass(string profilerTag, bool opaque, RenderPassEvent evt, RenderQueueRange renderQueueRange, LayerMask layerMask, StencilState stencilState, int stencilReference): this(profilerTag,new ShaderTagId[] { 
	    new ShaderTagId("SRPDefaultUnlit"), 
	    new ShaderTagId("UniversalForward"),
	    new ShaderTagId("UniversalForwardOnly"), 
	    new ShaderTagId("LightweightForward")},
        opaque, evt, renderQueueRange, layerMask, stencilState, stencilReference)

以及对于URP的多Pass的执行顺序并不是更加Pass在代码中的顺序，而是跟随上述代码中的枚举顺序。
如下
```CG
Shader "Unlit/NewUnlitShader2"
{
    Properties
    {
        _MainTex("Base(RGB)",2D)="white"{}
        _OutLineColor("_OutLineColor",Color)=(1,1,1,1)
    }
    SubShader
    {
        ZTest Off
        Tags
        {
            "RenderType"="Opaque"
        }
        LOD 100

        Pass
        {
            Name "Outline"
            Tags
            {
                "LightMode"="UniversalForward"
                "Queue"="2000"
            }

            Stencil
            {
                Ref 250
                Comp NotEqual
            }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include"Lighting.cginc"
            #include"UnityCG.cginc"

            sampler2D _MainTex;
            float4 _MainTex_ST;
            float4 _OutLineColor;

            struct v2f
            {
                float4 vertex:SV_POSITION;
            };

            struct appdata
            {
                float4 vertex:POSITION;
            };

            v2f vert(appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex*1.1);
                return o;
            }

            fixed4 frag(v2f i):SV_Target
            {
                return _OutLineColor;
            }
            ENDCG
        }
        Pass
        {
            Name "Self"
            Tags
            {
                "LightMode"="SRPDefaultUnlit"
                "Queue"="2000"
            }
            Stencil
            {
                Ref 250
                Comp Always
                Pass Replace
            }

            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include"Lighting.cginc"
            #include"UnityCG.cginc"

            sampler2D _MainTex;
            float4 _MainTex_ST;

            struct v2f
            {
                float4 vertex:SV_POSITION;
                float2 uv:TEXCOORD0;
            };

            struct appdata
            {
                float4 vertex:POSITION;
                float2 uv:TEXCOORD0;
            };

            v2f vert(appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                return o;
            }

            fixed4 frag(v2f i):SV_Target
            {
                fixed4 col = tex2D(_MainTex, i.uv);
                return col;
            }
            ENDCG
        }
    }
}
```
执行顺序为 self的渲染自身pass，然后渲染轮廓的outline pass

#### Stencil

	Stencil
	{
    	Ref <ref>
    	ReadMask <readMask>
    	WriteMask <writeMask>
    	Comp <comparisonOperation>
    	Pass <passOperation>
    	Fail <failOperation>
    	ZFail <zFailOperation>
    	CompBack <comparisonOperationBack>
    	PassBack <passOperationBack>
    	FailBack <failOperationBack>
    	ZFailBack <zFailOperationBack>
    	CompFront <comparisonOperationFront>
    	PassFront <passOperationFront>
    	FailFront <failOperationFront>
    	ZFailFront <zFailOperationFront>
	}
以上为shaderlab中的模板测试属性，ref为标准值，comp为比较方法，pass、fail、zfail为模板缓冲数据与标准值对比后的相应结果的操作，一般使用时如下
	
	Stencil
	{
		Ref [_refVal]
		Comp
		Pass
	}