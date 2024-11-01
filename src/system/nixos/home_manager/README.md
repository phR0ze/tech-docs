# Home Manager
Home Manager is the defacto standard for managing user applications and dot files. It was built as a 
companion to Nix but focused on user configuration. Although Nix and Home Manager's focus on a cross 
platform, multi-distro, multi-user type paradigm is super useful for those that use case fits it is 
less than ideal for the typical single user consumer type system. 

Single user consumer systems, my use case, typically want system wide user configuration where their 
single user and root user mostly mirror a similar configuration and most importantly installed 
applications are uniformly available to all users.

***WARNING*** I found home manager to be overly complicated to use and unnessary. I really don't want 
an additional daemon running in the background just to install user configuration. Instead I built a 
simple nix module that does some of the basics for me which is invoked on boot and checks for any 
changes needed.

<!-- 
vim: ts=2:sw=2:sts=2
-->
