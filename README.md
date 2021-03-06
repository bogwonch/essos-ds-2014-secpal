# Towards an authorization framework for app security checking
Joseph Hallett and David Aspinall

### Abstract

Apps don’t come with any guarantees that they are not malicious. This paper introduces a PhD project designing the authorization framework used for App Guarden. App Guarden is a new project that uses a flexible assurance framework based on distribution of evidence, attestation and checking algorithms to make explicit why an app isn’t dangerous and to allow users to describe how they want apps on their devices to behave. We use the SecPAL policy language to implement a device policy and give a brief example of a policy being used. Finally we use SecPAL to describe some of the differences between current app markets.

Introduction
============

App stores allow users to buy and install software on their devices with a minimum of fuss. Users trust the app stores not to supply them with malware but sometimes this trust is misplaced. A survey of the various different Android markets revealed some malware in all of them—including Google’s own Play Store Y. Zhou et al. (2012). Malware on mobile platforms typically steals user information and monetises itself by sending premium rate text messages Porter Felt et al. (2011). Sometimes malware on mobile platforms isn’t intentional: developers are not security specialists and sometimes make poor security decisions such as signing apps with the same key Barrera et al. (2012) which can lead to permission escalation on Android or by requesting excessive permissions Felt et al. (2011).

Both Google and Apple (who operate the two biggest app markets) check apps submitted to them for malicious code, however neither states exactly what they check for. There is expectation amongst users that these markets check for and are free from malware, but this trust is misplaced Enck and McDaniel (2010). Attempts to reverse engineer the checking procedures for Google’s store suggest that they incorporate static and dynamic analysis as well as manual checking by a human being Oberheide and Miller (2012). They offer no guarantees however that any check was actually run.

Digital evidence Stark (2009) can be used to confirm the results from static analyses; it is similar to older techniques G. Necula and Lee (1996) and allows us to split the *inferring* of a static analysis result (which may be computationally expensive) from the *checking* the evidence (which should be efficient). This allows us to verify the results of static analyses for security properties without having to fully run the analysis.

Static analysis checks used to create digital evidence could include taint analysis tools to check where sensitive information is being leaked between apps Enck et al. (2010), or a security proof based on the architecture of the machine Barthe et al. (2007).

The App Guarden project aims to use digital evidence to provide apps with guarantees that the app cannot act maliciously. This will allow us to make explicit the checks made on an app. With explicit checks users will be able to write policies which describe what kinds of apps they will allow on their devices, and what level of trust they require in these checks for an app to be considered *installable*. We aim to create an enhanced app store that uses inference services, and assurance logics on a modified Android to increase trust in devices.

Figure [fig:architecture] shows an App Guarden store. It provides apps with evidence generated by an inference service. Devices can check the evidence using a trusted checking service, or rely on their trust that the store believed the app to be safe. The SecPAL Engine on the phone is used to check the assertions made by the store and the checker service and enforce a device policy.

[fig:architecture]

Aims and Objectives
===================

Our aim is to create a modified Android ecosystem where apps come with digital evidence for security and where users can say how they want apps to behave on their device. To do this we will:

-   Show how an authorization logic can specify a policy for how a device behaves. This will use ideas such as proof checking and inference, and show how trust can be distributed between principals.

-   Show how to use the logic to specify a device policy. We will use digital evidence, and assertions between trusted principals and show how the trust relationships can be propagated between devices. We show how we can use these statements to build confidence that an app is secure.

-   Describe the differences between current app markets using the logic and show how they manage access to special resources (such as the address book or a user’s photos). This will allow us to compare App Guarden to other systems and allow comparison between other different schemes.

-   Put the logic in a real device and app store and see how the system behaves with real malware and how they interact with device policies. We will look for ways to attack devices with our framework and explore it’s limitations.

SecPAL and App Guarden
======================

Suppose a user has a mobile device. They might wish that:

> “No app installed on my phone will send my location to an advertiser, and I wont install anything that Google says is malware.”

We call this their *device policy*. It says what the user will allow on their device for an app to be installable. To formally write the device policies we use the SecPAL authorization language Becker, Fournet, and Gordon (2006). The device policy is show in ([sp:DevicePolicy]). SecPAL is designed for distributed access control decisions. It was designed to be human readable which makes it ideal for our application as we would like users to be able to understand (if perhaps not write) their own device policies. Another advantage is that it lets us separate the constraint solving (the digital evidence checking) from the authorization logic. This means there is no restriction on having the digital evidence be in the same format and logic as the assurance framework (as is the case with other authorization logics such as BLF Whitehead, Abadi, and Necula (2004)); this increases our flexibility for implementation as we can re-purpose existing logic solvers.

SecPAL is designed to be extensible. Authorization statements take the form of assertions made by principals. Statements can include predicates which are defined by the SecPAL program, as well as *constraints* (as seen in ([sp:NlliSaysEviShowsAppPol])) that can be checked as sub-procedures for the main query engine. We add the following statements:

\[\begin{aligned}
  e_\text{evidence}\textsf{~shows~}e_\text{app}\textsf{~meets~}e_\text{policy} \label{eqn:shows}\\
  e_\text{app}\textsf{~meets~}e_\text{policy} \label{eqn:meets}\end{aligned}\]

The…predicate in ([eqn:shows]) tells us that some evidence could be checked to show that an app satisfies a security policy; an example of which can be seen in ([sp:NlliSaysEviShowsAppPol]) bellow (and which also uses a constraint). The plainin ([eqn:meets]) is an assertion that an app meets a policy whilst offering no actual evidence that the app is secure. Thestatement is stronger than thestatement as anyone who says that some evidence shows an app meets a policy suggests they would also say the app meets policy. We can write this in SecPAL with the assertion: “\(\textrm{Phone}\textsf{~says~}\textsl{app}\textsf{~meets~}\textsl{policy}\textsf{~if~}\textsl{evidence}\textsf{~shows~}\textsl{app}\textsf{~meets~}\textsl{policy}\).”

We also add an statement to indicate an app is installable on a device and a statement to say an app needs a permission to run.

SecPAL allows for attestation of said statements with the *can say* statement: these come with a delegation depth. A delegation depth of zero (as shown in ([sp:NlliCanSayNll])) means the statement must be spoken by the principal and given to us directly in the assertion context. A delegation depth of infinity (as in ([sp:GoogleCanSayNm])) means that the statement might not be given to us directly but rather inferred from a statement by different principal but with several *can say* phrases linking it back to the required principal.

To evaluate the device policy and decide whether a specific app is installable SecPAL requires a set of assertions called the *assertion-context*. An example assertion-context and the proof, using two of SecPAL’s evaluation rules from the original paper Becker, Fournet, and Gordon (2006). To illustrate a usage of SecPAL we give an example where a user wishes to decide whether an app can be installed. To do this we use the assertion context bellow:

\[\begin{aligned}
    AC & \coloneqq \left\{ \right. \nonumber\\
    \textrm{Phone}&\textsf{~says~}\textsl{app}\textsf{~is installable}\nonumber\\
    \textsf{~if~}&\textsl{app}\textsf{~meets~}\textrm{NotMalware}, \nonumber\\
            &\textsl{app}\textsf{~meets~}\textrm{NoInfoLeaks}.
  \label{sp:DevicePolicy}\\
    \textrm{Phone}&\textsf{~says~}\textsl{app}\textsf{~meets~}\textsl{policy}\textsf{~if~}\textsl{evidence}\textsf{~shows~}\textsl{app}\textsf{~meets~}\textsl{policy}.
  \label{sp:showsmeets}\\
    \textrm{Phone}&\textsf{~says~}\textrm{NILInferer}\textsf{~can say}_{0}~\textsl{app}\textsf{~meets~}\textrm{NoInfoLeaks}.
  \label{sp:NlliCanSayNll}\\
    \textrm{Phone}&\textsf{~says~}\textrm{Google}\textsf{~can say}_{\infty}~
    \textsl{app}\textsf{~meets~}\textrm{NotMalware}.
  \label{sp:GoogleCanSayNm}\\
    \textrm{Google}&\textsf{~says~}\textrm{AVChecker}\textsf{~can say}_{0}~\textsl{app}\textsf{~meets~}\textrm{NotMalware}.
  \label{sp:AvcCanSayNm}\\
    \textrm{AVChecker}&\textsf{~says~}\textrm{Game}\textsf{~meets~}\textrm{NotMalware}.
  \label{sp:AvcSaysNm}\\
    \textrm{NILInferer}&\textsf{~says~}\textrm{Evidence}\textsf{~shows~}\textrm{Game}\textsf{~meets~}\textrm{NoInfoLeaks}\nonumber\\
         & \textsf{~if~}\textsf{LocalNILCheck}{\left(\textrm{Evidence},  \textrm{Game}\right)}=\textsf{true}.
  \label{sp:NlliSaysEviShowsAppPol}\\
  \left.\right\}\nonumber\end{aligned}\]

We evaluate it using the SecPAL rules shown in the cond ([rule:cond]) and can say ([rule:cansay]) rules. The cond rule is SecPAL’s equivalent to the *modus ponens* rule. Whereas can say is similar to *speaks for* relationships Abadi (2003): if a first principal says a second principal can say something and the second does say it then the first says it too. \(AC\) represents the assertion context, \(D\) is the current delegation level. From these rules and the assertion context we can prove an assertion that “\(\textrm{Phone}\textsf{~says~}\textrm{Game}\textsf{~is installable}\).”

\[\scalebox{0.8}{  \infer[\textsc{\small cond }]{    \left\{A\textsf{~says~}\textsl{fact}\textsf{~if~}\textsl{fact}_1,  \dots\textsl{fact}_n,  c\right\}\cup AC,  D \models
      A\textsf{~says~}\textsl{fact}}{    \begin{split}
    \left\{A\textsf{~says~}\textsl{fact}\textsf{~if~}\textsl{fact}_1,  \dots,  \textsl{fact}_n,  c\right\}\cup AC,  D \models
      A\textsf{~says~}\textsl{fact}_1,  \dots,  \textsl{fact}_n \\
    \models c \\
    vars(\textsl{fact}) = \emptyset
    \end{split}
  }}
  \label{rule:cond}\\\]

\[\scalebox{0.8}{  \infer[\textsc{\small can say}]{    AC,  \infty\models A\textsf{~says~}\textsl{fact}}{    AC,  \infty\models A\textsf{~says~}B\textsf{~can say}_{D}~\textsl{fact}&
    AC,  D\models B\textsf{~says~}\textsl{fact}}}
  \label{rule:cansay}\]

Comparing different devices
===========================

We can use the SecPAL language to describe the differences between various app markets and devices. The two most popular operating systems are Google’s Android, and Apple’s <span>iOS</span>. Apple’s offering is usually considered to be the more restrictive system as it doesn’t allow the use of alternate market places, whereas an Android user can choose to relax the restrictions and install from anywhere.

For example on iOS an app is only installable on an iPhone if Apple has said the app can be installed ([oas:iphoneinstall]). In contrast on Android any app can be installed if the user says so ([oas:anduser]). The user also trusts any app available on Google’s Play Store ([oas:androidinstall]), but the zero delegation-depth means that the user will only believe the app is installable if Google provides the assertion directly whereas the user in ([oas:anduser]) is free to delegate the decision because of the infinite delegation depth.

\[\begin{aligned}
  \textrm{iPhone}&\textsf{~says~}\textrm{Apple}\textsf{~can say}_{\infty}~\textsl{app}\textsf{~is installable}.
  \label{oas:iphoneinstall}\\
  \\
  \textrm{AndroidPhone}&\textsf{~says~}\textrm{AndroidUser}\textsf{~can say}_{\infty}~\textsl{app}\textsf{~is installable}.\label{oas:anduser}\\
  \textrm{AndroidUser}&\textsf{~says~}\textrm{Google}\textsf{~can say}_{0}~\textsl{app}\textsf{~is installable}.
  \label{oas:androidinstall}\end{aligned}\]

Similarly comparisons can be made when apps try to access resources. When accessing components like the address book or photos; on iOS this is only allowed when the user has explicitly okayed it. Other operating systems such as Windows Mobile also follow this pattern. On Android, however, an app can access any resource it declared itself as *requiring* it when it was installed.

\[\begin{aligned}
  \textrm{iPhone}&\textsf{~says~}\textrm{User}\textsf{~can say}_{0}~\textsl{app}\textsf{~can access~}\textrm{resource}.\\
  \\
  \textrm{AndroidPhone}&\textsf{~says~}\textsl{app}\textsf{~can access~}\textrm{resource} \nonumber \\
  \textsf{~if~}&\textsl{app}\textsf{~is installable}, \nonumber\\
  &\textsl{app}\textsf{~requires~}\textrm{resource}.\end{aligned}\]

This is a brief comparison of the implicit device policies in current markets. It shows however that SecPAL is capable of describing the various difference. This could be extended in future to allow proper comparisons of different markets.

Conclusion and future directions
================================

We have given a brief introduction to App Guarden and the SecPAL language used to describe policies. Using the SecPAL language we described some of the differences between current systems. The PhD work here is at an early stage. Next steps will include porting the SecPAL language to Android and using it to find what kinds of device policies are most effective at stopping dangerous apps, without overly inconveniencing the user. From here we will build an *enhanced app market* with its own set of policies where apps come with digital evidence to allow users to trust apps further.

Mobile devices are a growing target for malware, and with app stores appearing in conventional computers the differences between mobile and traditional computing environments are blurring. The App Guarden will help give users greater assurance their software is secure; and provide a new security mechanism to compliment access control and anti-malware techniques.

Abadi, M. 2003. “Logic in Access Control.” *18th Annual IEEE Symposium on Logic in Computer Science*: 228–233.

Barrera, D., J. Clark, D. McCarney, and P.C. van Oorschot. 2012. “Understanding and Improving App Installation Security Mechanisms Through Empirical Analysis of Android.” In *Security and Privacy in Smartphones and Mobile Devices*, 81–12. New York, New York, USA: ACM Press.

Barthe, G., L. Beringer, P. Crégut, B. Grégoire, M. Hofmann, P. Müller, E. Poll, G. Puebla, I. Stark, and E.ric Vétillard. 2007. “MOBIUS: Mobility, Ubiquity, Security.” *Trustworthy Global Computing* 4661 (Chapter 2): 10–29.

Becker, M.Y., C. Fournet, and A.D. Gordon. 2006. “SecPAL: Design and Semantics of a Decentralized Authorization Language.” Microsoft Research.

Enck, W., and P. McDaniel. 2010. “Not So Great Expectations.” *Secure Systems* (September): 1–3.

Enck, W., P. Gilbert, B.G. Chun, L.P. Cox, and J. Jung. 2010. “TaintDroid: An Information-Flow Tracking System for Realtime Privacy Monitoring on Smartphones.” *OSDI*.

Felt, A.P., E. Chin, S. Hanna, D. Song, and D. Wagner. 2011. “Android Permissions Demystified.” *Computer and Communications Security* (October): 627–638.

Necula, G.C., and P. Lee. 1996. “Proof-Carrying Code.” Carnegie Mellon University.

Oberheide, J., and C. Miller. 2012. “Dissecting the Android Bouncer.” *SummerCon*.

Porter Felt, A., M. Finifter, E. Chin, and S. Hanna. 2011. “A Survey of Mobile Malware in the Wild.” *Proceedings of the 1st Workshop on Security and Privacy in Smartphones and Mobile Devices, CCS-SPSM’11*.

Stark, I. 2009. “Reasons to Believe: Digital Evidence to Guarantee Trustworthy Mobile Code.” In *The European FET Conference*, 1–17.

Whitehead, N., M. Abadi, and G. Necula. 2004. “By Reason and Authority: a System for Authorization of Proof-carrying Code.” In *Computer Security Foundations Workshop, 2004. Proceedings. 17th IEEE*, 236–250. IEEE.

Zhou, Y., Z. Wang, W. Zhou, and X. Jiang. 2012. “Hey, You, Get Off of My Market: Detecting Malicious Apps in Official and Alternative Android Markets.” *Proceedings of the 19th Annual Symposium on Network and Distributed System Security*.


