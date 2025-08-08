---
title: "Get started with Neurodesk"
linkTitle: "Getting Started"
weight: 1
menu:
  main:
    weight: 9
alias: 
- /docs/getting-started/
description: >
---

<section id="startup" class="row -bg-light justify-content-left h-auto col-big-desktop">
	<div class="td-box">
		<p class="lead mt-2">Select your setups and follow further instructions in the provided link.
		</p>
	</div>
	<div class="container-fluid quick-starts">
		<div class="container">
			<div class="row">
				<div class="col-md-10 start-locally-col">
					<div class="row">
						<div class="col-md-3 headings">
							<div class="col-md-12 title-block">
								<div class="option-text">Compute Platform</div>
							</div>
							<div class="col-md-12 title-block">
								<div class="option-text">Your OS</div>
							</div>
							<div class="col-md-12 title-block">
								<div class="option-text">Interface</div>
							</div>
							<div class="col-md-12 title-block">
								<div class="option-text">Processor</div>
							</div>
							<div class="col-md-12 title-block command-block">
								<div class="option-text command-text">Instructions:</div>
							</div>
						</div>
						<div class="col-md-9">
							<div class="row platform">
								<div class="col-md-12 title-block mobile-heading">
									<div class="option-text">Compute Platform</div>
								</div>
								<div class="col-md-3 option block version" id="local">
									<div class="option-text">Local PC</div>
								</div>
								<div class="col-md-3 option block version" id="hpc">
									<div class="option-text">HPC</div>
								</div>
								<div class="col-md-3 option block version" id="cloud">
									<div class="option-text">Cloud</div>
								</div>
								<div class="col-md-3 option block version" id="colab">
									<div class="option-text">Google Colab</div>
								</div>
							</div>
							<div class="row os">
								<div class="col-md-12 title-block mobile-heading">
									<div class="option-text">Operating System</div>
								</div>
								<div class="col-md-4 option block" id="linux">
									<div class="option-text">Linux</div>
								</div>
								<div class="col-md-4 option block" id="macos">
									<div class="option-text">Mac</div>
								</div>
								<div class="col-md-4 option block" id="windows">
									<div class="option-text">Windows</div>
								</div>
							</div>
							<div class="row interface">
								<div class="col-md-12 title-block mobile-heading">
									<div class="option-text">Interface</div>
								</div>
								<div class="col-md-3 option block version selected" id="gui">
									<div class="option-text">Desktop</div>
								</div>
								<div class="col-md-3 option block version" id="cmd">
									<div class="option-text">Command Line</div>
								</div>
								<div class="col-md-3 option block version" id="container">
									<div class="option-text">Container</div>
								</div>
								<div class="col-md-3 option block version" id="vscode">
									<div class="option-text">VSCode</div>
								</div>
							</div>
							<div class="row processor">
								<div class="col-md-12 title-block mobile-heading">
									<div class="option-text">Processor</div>
								</div>
								<div class="col-md-4 option block" id="x86">
									<div class="option-text">x86</div>
								</div>
								<div class="col-md-4 option block" id="arm">
									<div class="option-text">ARM</div>
								</div>
								<div class="col-md-4 option block" id="gpu">
									<div class="option-text">GPU</div>
								</div>
							</div>
							<div class="row instruction">
								<div class="col-md-12 title-block command-mobile-heading">
									<div class="option-text">Instructions:</div>
								</div>
								<div class="command-container">
									<div class="col-md-12" id="command">
									</div>
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</section>
<script src="/static/js/command.js"></script>

## Release
Ensure you note the release date of the Neurodesktop container image during installation, as it is indicated in the _docker run_ command, e.g. 

![version](/static/docs/getting-started/neurodeskapp/version.png 'version')

We regularly update Neurodesktop for optimal performance and updated software. Check the [Release History](/docs/overview/release-history) for past releases. To upgrade your container, replace the release number with your desired version. For replicable analysis pipelines, share the stable release number you used, enabling others to recreate your work environment anywhere.

## Video tutorial
See below for a 14 minute tutorial on getting started. 
{{< youtube 2ATgTOsiGdY >}}
<!-- Click [here](https://www.youtube.com/watch?v=2ATgTOsiGdY&list=PLXHdMkqf4kf_ch9quScSTX8YYaSnqnmqX&index=6) to watch a 14 minute tutorial video from eResearch 2021 -->

You can also view this PowerPoint presentation that includes dynamic navigation and interactive buttons: [Neurodesk Overview SlideShow](https://osf.io/f2n7a) - and [PDF version of the presentation](https://osf.io/ca8rw)