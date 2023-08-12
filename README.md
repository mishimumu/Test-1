# Test
hhhh
mishimumu modify
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Spine.Unity;
using Spine;
using System;

public class SpineHelp : MonoBehaviour {
	public SkeletonGraphic[] skeletonGraphics;
	public SkeletonAnimation[] skeletonAnimations;
    public BoneFollowerGraphic[] boneFollowerGraphics;
	public ParticleSystemZoomHelp[] particleSystemZoomHelps;
	public Action AnimationEndEvent;
	public bool loop = false;
	public float duration = 0;

	private int playAnimationTime = 1;//动画播放次数
	private int currentPlayAnimationTime = 0;
	private bool isDurationSet = false;

	private int animationCounts = 0;

	public Animator[] animators;
	public ParticleSystem[] particleSystems;
	// Use this for initialization
	void Awake () {
		RoomProgressHelp.Instance.StartPassEvent += Play;
		RoomProgressHelp.Instance.PausePassEvent += pasue;
	}

	public void pasue()
	{
		if (skeletonAnimations != null)
		{
			for (int i = 0; i < skeletonAnimations.Length; i++)
			{
				skeletonAnimations[i].AnimationState.TimeScale = 0;
			}
		}
		if (skeletonGraphics != null)
		{
			for (int i = 0; i < skeletonGraphics.Length; i++)
			{
				skeletonGraphics[i].AnimationState.TimeScale = 0;
			}
		}
		if (animators != null)
		{
			for (int i = 0; i < animators.Length; i++)
			{
				animators[i].speed = 0;
			}
		}
		if (particleSystems != null)
		{
			for (int i = 0; i < particleSystems.Length; i++)
			{
				particleSystems[i].Pause();
			}
		}
	}

	public void Play()
	{
		if (skeletonAnimations != null)
		{
			for (int i = 0; i < skeletonAnimations.Length; i++)
			{
				skeletonAnimations[i].AnimationState.TimeScale = 1;
			}
		}
		if (skeletonGraphics != null)
		{
			for (int i = 0; i < skeletonGraphics.Length; i++)
			{
				skeletonGraphics[i].AnimationState.TimeScale = 1;
			}
		}
		if (animators != null)
		{
			for (int i = 0; i < animators.Length; i++)
			{
				animators[i].speed = 1;
			}
		}
		if (particleSystems != null)
		{
			for (int i = 0; i < particleSystems.Length; i++)
			{
				particleSystems[i].Play();
			}
		}
	}

	
	public void SetAnimationDuration()
	{
		if(isDurationSet){
			return;
		}
		isDurationSet = true;
		skeletonGraphics = gameObject.GetComponentsInChildren<SkeletonGraphic>();
		skeletonAnimations = gameObject.GetComponentsInChildren<SkeletonAnimation>();
		particleSystems = gameObject.GetComponentsInChildren<ParticleSystem>();
		animators = gameObject.GetComponentsInChildren<Animator>();
        boneFollowerGraphics = gameObject.GetComponentsInChildren<BoneFollowerGraphic>();
		particleSystemZoomHelps= gameObject.GetComponentsInChildren<ParticleSystemZoomHelp>();

		if (skeletonAnimations != null)
		{
			for (int i = 0; i < skeletonAnimations.Length; i++)
			{
				skeletonAnimations[i].loop = false;
				duration = Math.Max(duration,skeletonAnimations[i].skeletonDataAsset.GetSkeletonData(false).FindAnimation(skeletonAnimations[i].AnimationName).Duration);
			}
		}
		if (skeletonGraphics != null)
		{
			for (int i = 0; i < skeletonGraphics.Length; i++)
			{
				skeletonGraphics[i].startingLoop = false;
				duration = Math.Max(duration,skeletonGraphics[i].skeletonDataAsset.GetSkeletonData(false).FindAnimation(skeletonGraphics[i].startingAnimation).Duration);
			}
		}

	
	}

	public void Play(bool isLoop = true)
	{
		loop = isLoop;
		int index = 0;
		if (skeletonAnimations != null)
		{
			for (int i = 0; i < skeletonAnimations.Length; i++)
			{
				SetAnimation(skeletonAnimations[i],i,skeletonAnimations[i].AnimationName, isLoop);
				index ++;
			}
		}
		if (skeletonGraphics != null)
		{
			for (int i = index; i < skeletonGraphics.Length; i++)
			{
				SetAnimation(skeletonGraphics[i],i,skeletonGraphics[i].startingAnimation, isLoop);
			}
		}
        if (boneFollowerGraphics != null)
        {
            for (int i = 0; i < boneFollowerGraphics.Length; i++)
            {
                boneFollowerGraphics[i].Initialize();
            }
        }
		if (particleSystemZoomHelps != null)
		{
			for (int i = 0; i < particleSystemZoomHelps.Length; i++)
			{
				particleSystemZoomHelps[i].ResetActive();
			}
		}

        currentPlayAnimationTime ++;
		if (RoomProgressHelp.Instance.isPause)
		{
			pasue();
		}
	}

	public void ResetPlayAnimationTime(int playTime)
	{
		playAnimationTime = playTime;
		if(playAnimationTime < 1){
			playAnimationTime = 1;
		}
		currentPlayAnimationTime = 0;
		animationCounts = 0;
	}

	void OnEnable()
	{
		
		
	}
	void OnDestroy()
	{
		RoomProgressHelp.Instance.StartPassEvent -= Play;
		RoomProgressHelp.Instance.PausePassEvent -= pasue;

	}

	void OnDisable()
	{
		if (skeletonAnimations != null)
		{
			for (int i = 0; i < skeletonAnimations.Length; i++)
			{
				skeletonAnimations[i].AnimationState.End -= EndEvent;
			}
		}
		if (skeletonGraphics != null)
		{
			for (int i = 0; i < skeletonGraphics.Length; i++)
			{
				skeletonGraphics[i].AnimationState.End -= EndEvent;
			}
		}
		animationCounts = 0;
	}

	private void EndEvent(TrackEntry trackEntry)
	{
        animationCounts --;
        if (currentPlayAnimationTime < playAnimationTime && animationCounts <= 0)
        {
            Play(this.loop);
        }
        if (AnimationEndEvent != null && currentPlayAnimationTime == playAnimationTime && animationCounts <= 0)
        {
                AnimationEndEvent();
        }
		
	}

	void SetAnimation(SkeletonGraphic skeletonGraphic,int trackIndex,string actionname, bool isLoop)
	{
		if (skeletonGraphic != null)
		{
			skeletonGraphic.Initialize(true);
			TrackEntry entry = skeletonGraphic.AnimationState.SetAnimation(0, actionname, isLoop);
			animationCounts ++;
			skeletonGraphic.AnimationState.Complete += EndEvent;
		}
	}
	void SetAnimation(SkeletonAnimation skeletonAnimation,int trackIndex, string actionname, bool isLoop)
	{
		if (skeletonAnimation != null)
		{
			skeletonAnimation.Initialize(true);
			TrackEntry entry = skeletonAnimation.AnimationState.SetAnimation(0, actionname, isLoop);
			animationCounts ++;
			skeletonAnimation.AnimationState.Complete += EndEvent;
		}
	}

}


