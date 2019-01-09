## Xtra. Parallel and stashing
Now we have two processes that actually can be run in parallel. The `build` and `javadoc` steps both take in the sourcecode and produces artifacts. So lets try to run them in parallel.

> This assignment is loosely formulated, so you need to [look things up yourself](https://jenkins.io/doc/pipeline/steps/) in order to complete this one

* Stash the source code cloned in `Preparation` and call it source
* `build` and `javadoc` steps needs to be included in a parallel step like the one below

```groovy
def builders = [
	"build": {
		node {}
	},
	"javadoc": {
	node {}
	}
]
stage('parallel'){
	parallel builders
}
```

* Unstash the source code in both stages, and perform the normal build steps
* Stash the results instead of archiving. Call them `jar` and `javadoc`
* Unstash them in the `Results` step in the end where you archive them.