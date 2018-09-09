# This file contains various how-tos intended to aid in FreeBSD ports creation.

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

