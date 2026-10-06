"""Test suite for tap-github."""

import requests
import requests_cache
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

requests_cache.install_cache(
    ".cache/api_calls_tests_cache",
    backend="sqlite",
    ignored_parameters=["Authorization", "User-Agent", "If-modified-since"],
    match_headers=True,
    expire_after=24 * 60 * 60,
    allowable_methods=["GET", "POST"],
)

_retry_strategy = Retry(
    total=3,
    status_forcelist=[403, 429],
    backoff_factor=2,
    respect_retry_after_header=True,
)
_retry_adapter = HTTPAdapter(max_retries=_retry_strategy)

_original_session_init = requests.Session.__init__


def _session_init_with_retries(self, *args, **kwargs):
    _original_session_init(self, *args, **kwargs)
    self.mount("https://", _retry_adapter)
    self.mount("http://", _retry_adapter)


requests.Session.__init__ = _session_init_with_retries
