# Various how-tos intended to aid in FreeBSD ports creation

## The project doesn't provide tarballs, and only offers the Git repository. How to fetch such code?

You can use the following ```do-fetch``` rule:

```
do-fetch:
	@if [ "${FORCE_FETCH_ALL}" = "true" ] || ! [ -f "${DISTDIR}/${DIST_SUBDIR}/${DISTNAME}${EXTRACT_SUFX}" ]; then \
	  ${MKDIR} ${DISTDIR}/${DIST_SUBDIR} && \
	  cd ${DISTDIR}/${DIST_SUBDIR} && \
	    git clone -q ${GIT_URL} ${PORTNAME}-${DISTVERSIONFULL} && \
	    (cd ${PORTNAME}-${DISTVERSIONFULL} && git reset -q --hard ${DISTVERSIONFULL} && ${RM} -r .git) && \
	    ${FIND} ${PORTNAME}-${DISTVERSIONFULL} -and -exec ${TOUCH} -h -d 1970-01-01T00:00:00Z {} \; && \
	    ${FIND} ${PORTNAME}-${DISTVERSIONFULL} -print0 | LC_ALL=C ${SORT} -z | \
	        ${TAR} czf ${PORTNAME}-${DISTVERSIONFULL}${EXTRACT_SUFX} --format=bsdtar --uid 0 --gid 0 --options gzip:!timestamp --no-recursion --null -T - && \
	    ${RM} -r ${PORTNAME}-${DISTVERSIONFULL}; \
	fi
```

It depends on the variables:
* ```GIT_URL```: The URL of the remote Git repository.
* ```DISTVERSION```: The version that you wish to use.
* ```DISTVERSIONPREFIX```: The version prefix that the Git tag is defined with.
* ```DISTVERSIONSUFFIX```: The version suffix that the Git tag is defined with.

The latter three variables are converted to ```DISTVERSIONFULL``` by the framework, and ```DISTVERSIONFULL``` is used here.


## The project doesn't provide tarballs, and only offers the Subversion repository. How to fetch such code?

You can use the following ```do-fetch``` rule:

```
do-fetch:
	@if [ "${FORCE_FETCH_ALL}" = "true" ] || ! [ -f "${DISTDIR}/${DIST_SUBDIR}/${DISTNAME}${EXTRACT_SUFX}" ]; then \
	  ${MKDIR} ${DISTDIR}/${DIST_SUBDIR} && \
	  cd ${DISTDIR}/${DIST_SUBDIR} && \
	    svn co -r ${SVN_REVISION} ${SVN_URL} ${PORTNAME}-${DISTVERSIONFULL} && \
	    (cd ${PORTNAME}-${DISTVERSIONFULL} && ${RM} -r .svn) && \
	    ${FIND} ${PORTNAME}-${DISTVERSIONFULL} -and -exec ${TOUCH} -h -d 1970-01-01T00:00:00Z {} \; && \
	    ${FIND} ${PORTNAME}-${DISTVERSIONFULL} -print0 | LC_ALL=C ${SORT} -z | \
	        ${TAR} czf ${PORTNAME}-${DISTVERSIONFULL}${EXTRACT_SUFX} --format=bsdtar --uid 0 --gid 0 --options gzip:!timestamp --no-recursion --null -T - && \
	    ${RM} -r ${PORTNAME}-${DISTVERSIONFULL}; \
	fi
```

It depends on the variables:
* ```SVN_URL```: The URL of the remote Subversion repository.
* ```DISTVERSION```: The version that you wish to use.
* ```DISTVERSIONPREFIX```: The version prefix that the Git tag is defined with.
* ```DISTVERSIONSUFFIX```: The version suffix that the Git tag is defined with.

The latter three variables are converted to ```DISTVERSIONFULL``` by the framework, and ```DISTVERSIONFULL``` is used here.


## How to get the Git identifier of the remote Git repository?

You can use this script ```git-remote-describe-tag```:
```
#!/bin/sh

REPO=$1
REV=$2

DIR=$(echo $1 | sed -e 's|.*/|| ; s|\.git$||')

(
  git clone $REPO &&
  (cd $DIR && git describe --tags $2);
  rm -rf $DIR
)
```

Save this code into the file git-remote-describe, and invoke it as:
```
git-remote-describe-tag {git-url} [{git-tag}]
```

It prints the tag, when possible, i.e. when the repository has a release tag:
```
$ git-remote-describe-tag git@github.com:freebsd/poudriere.git
Cloning into 'poudriere'...
remote: Counting objects: 38672, done.
remote: Total 38672 (delta 0), reused 0 (delta 0), pack-reused 38672
Receiving objects: 100% (38672/38672), 13.03 MiB | 6.06 MiB/s, done.
Resolving deltas: 100% (23910/23910), done.
3.2.7-391-gf1e9f1d6
```

The ```3.2.7-391``` part should be put in ```DISTVERSION```, and the ```-gf1e9f1d6``` part should be put in ```DISTVERSIONSUFFIX```.

## How to run tests in a cmake-based port?

Let's assume that the project enables tests with ```BUILD_TESTS``` cmake variable, and runs them with ```test``` target.

You should first disable tests for build:
```
CMAKE_OFF+=BUILD_TESTS
```
and then enable tests by creating the ```do-test``` target like this:
```
do-test:
	@cd ${BUILD_WRKSRC} && \
		${SETENV} ${CONFIGURE_ENV} ${CMAKE_BIN} ${CMAKE_ARGS} -DBUILD_TESTS=ON ${CMAKE_SOURCE_PATH} && \
		${SETENV} ${MAKE_ENV} ${MAKE_CMD} ${MAKE_ARGS} ${ALL_TARGET} && \
		${SETENV} ${MAKE_ENV} ${MAKE_CMD} ${MAKE_ARGS} test
```
```make test``` will first re-configure the project with tests enabled, then build them, and then will run them.

This approach is better than creating the ```TEST``` port option, because testing should only happen during build,
and once the port is installed users don't need to see the ```TEST``` option.

