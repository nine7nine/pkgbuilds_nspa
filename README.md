# pkgbuilds_nspa

Git Repo for my wine / linux flavours for Archlinux

  Linux-NSPA; my custom kernel. 

- contains futex-multiple-wait patch(actually, 2 versions). needed for wine+fsync.
- some of intel's patchwork. 
- gcc optimization
- the percpu-rwsem rewrite (lives in -rt 5.6.x, as well).
- swait stuff from -rt. (not really -rt specifc though. in either case.).
- a couple fixes for pesky issues (for me).

set to 1000hz. I use threaded interrupts. etc. not using preempt-rt right now.

mainly posting for backup, but also for anyone wanting to try my wine-nspa builds, will need
kernel support. so the futex-multiple-wait support is available. 5.2.x/5.5 diff version

i also only 'loosely' follow Arch kernel updates. I don't need to chase updates.

  Wine-NSPA; wine geared for proaudio. (*WIP*, like pre-alpha).
  
- Built on Wine-Staging
- Fsync support aka: hybrid synchronization in Wine. (reuires kernel support).

  Enabled With:
 
  * WINEFSYNC_SPINCOUNT=128
  * WINESYNC=1 (0 - disables).

- Initial implmentation of PROCESS_PRIOCLASS_* combined with implementing; 
  PROCESS_PRIOCLASS_REALTIME thread prorities mapped to SCHED_FIFO (with some revisions).

  Example:
  
  * env WINE_RT_PRIO=70 wine foo
  
  unlike wine-staging - I don't allow setting the wineserver thread independently of the base priority.
  Another difference; WINE_RT_PRIO is a MAX value and decrements; wineserver=>any apx futex RT threads and finally
  the realtime process threads (ie: an app's high priority threads).
  
  the (proportional) step between the above threads is set within wine, so it always gets it right, regardless
  of the WINE_RT_PRIO value.
  
  NOTE: while this certainly helps apps, ideally -- the legacy set-realtime-without-wineserver.patch needs
  to be ported / re-implemented using futex-multiple-wait. The arguably most important detail in that patch is
  how it was not only able to set realtime priorites - but actually move threads completely out of wineserver,
  meaning they did not wait on user threads. These threads were tagged/hooked in SetThreadPriority() (WinAPI),
  then caught in ntdll and handled differently. (just look at the functions in the patch's ntdll and kernel32 parts).
  
  I'm probably not smart enough to re-implement this, but I'm stubborn -- so we shall see. lol regardless, I am
  correct that a re-implmenetation of this patch would be killer -- as we could hook these threads, which would
  get rid of a lot of traffic in wineserver.

- I also use these staging settings.
  
  * STAGING_SHARED_MEMORY=1
  * STAGING_WRITECOPY=1

  Wine-NSPA contains some other stuff, as well.
  
  this build should make winelib / bridge and VST people happy. Not sure about the 
  DAW scenario, beyond testing thread priorities in Reaper + running Kontakt/Reaktor.
  Unfortunately, DAWs are more complex and the issues are harder to solve. 
  
  We can't handle wait-on-multiple-objects in wine, due to wineserver 
  being single-threaded and a limitation of hybrid synchronization. It's possible
  a DAW might run better - it's possible it's perfomrance may be hurt, or there could be
  threading issues. 
  
  lots more TODO/WIP.
