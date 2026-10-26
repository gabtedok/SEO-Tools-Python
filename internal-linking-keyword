import requests
from bs4 import BeautifulSoup
import networkx as nx
import matplotlib.pyplot as plt
import re
from urllib.parse import urljoin, urlparse, urlunparse


def clean_url(url):
    parsed = urlparse(url)

    return urlunparse((
        parsed.scheme,
        parsed.netloc,
        parsed.path.rstrip("/"),
        "",
        "",
        ""
    ))


def display_url(url):
    """
    URL que se muestra en el gráfico.
    Quita https://, http:// y www.
    """

    url = re.sub(r"^https?://", "", url, flags=re.IGNORECASE)
    url = re.sub(r"^www\.", "", url, flags=re.IGNORECASE)

    return url


def is_menu_link(tag):
    """
    Detecta enlaces dentro de menú, navegación,
    header o mega-menu.
    """

    parent = tag

    while parent:

        if getattr(parent, "name", None) in ["nav", "header"]:
            return True

        if hasattr(parent, "get"):

            classes = parent.get("class", [])
            element_id = parent.get("id", "")
            role = parent.get("role", "")

            classes_text = " ".join(classes).lower()
            id_text = str(element_id).lower()
            role_text = str(role).lower()

            menu_terms = [
                "menu",
                "nav",
                "navbar",
                "navigation",
                "mega-menu",
                "megamenu",
                "header",
                "main-menu",
                "primary-menu"
            ]

            for term in menu_terms:

                if (
                    term in classes_text
                    or term in id_text
                    or term in role_text
                ):
                    return True

        parent = parent.parent

    return False


def get_links_from_url(url, base_url):

    try:

        response = requests.get(
            url,
            timeout=15,
            headers={
                "User-Agent": (
                    "Mozilla/5.0 "
                    "(Windows NT 10.0; Win64; x64) "
                    "AppleWebKit/537.36 "
                    "(KHTML, like Gecko) "
                    "Chrome/120 Safari/537.36"
                )
            }
        )

        response.raise_for_status()

        soup = BeautifulSoup(
            response.content,
            "html.parser"
        )

        links = set()

        for link in soup.find_all("a", href=True):

            # EXCLUIR MENÚ
            if is_menu_link(link):
                continue

            href = urljoin(
                url,
                link["href"]
            )

            href = clean_url(href)

            # SOLO ENLACES INTERNOS
            if (
                urlparse(href).netloc
                != urlparse(base_url).netloc
            ):
                continue

            # SOLO URLs CON AVORIAZ
            if re.search(
                r"avoriaz",
                href,
                re.IGNORECASE
            ):
                links.add(href)

        return links

    except Exception as e:

        print(
            f"Error crawling {url}: {e}"
        )

        return set()


def crawl_website(
    base_url,
    max_depth=2
):

    base_url = clean_url(
        base_url
    )

    crawled = set()

    graph = nx.DiGraph()

    to_crawl = [
        (base_url, 0)
    ]

    while to_crawl:

        current_url, depth = (
            to_crawl.pop(0)
        )

        if current_url in crawled:
            continue

        if depth > max_depth:
            continue

        crawled.add(
            current_url
        )

        print(
            f"Crawling depth {depth}: "
            f"{current_url}"
        )

        graph.add_node(
            current_url
        )

        links = get_links_from_url(
            current_url,
            base_url
        )

        for link in links:

            if link == current_url:
                continue

            graph.add_edge(
                current_url,
                link
            )

            if link not in crawled:

                to_crawl.append(
                    (
                        link,
                        depth + 1
                    )
                )

    return graph


def visualize_graph(graph):

    if graph.number_of_edges() == 0:

        print()
        print(
            "No se encontraron enlaces suficientes."
        )

        return

    # GRÁFICO MÁS GRANDE
    plt.figure(
        figsize=(30, 20)
    )

    pos = nx.spring_layout(
        graph,
        seed=42,

        # Más separación entre nodos
        k=4.5,

        iterations=250
    )

    node_sizes = []

    for node in graph.nodes():

        incoming = graph.in_degree(
            node
        )

        outgoing = graph.out_degree(
            node
        )

        size = (
            1500
            + incoming * 800
            + outgoing * 400
        )

        node_sizes.append(
            size
        )

    # URL SIN HTTPS://WWW.
    labels = {
        node: display_url(node)
        for node in graph.nodes()
    }

    nx.draw_networkx_nodes(
        graph,
        pos,
        node_size=node_sizes,
        alpha=0.85
    )

    nx.draw_networkx_edges(
        graph,
        pos,
        arrows=True,
        arrowsize=22,
        width=1.2,
        alpha=0.35,
        connectionstyle="arc3,rad=0.05"
    )

    # TEXTO MÁS GRANDE
    nx.draw_networkx_labels(
        graph,
        pos,
        labels=labels,

        # Antes era 6
        font_size=10,

        font_weight="bold"
    )

    plt.title(
        "Internal Linking Map — Avoriaz",
        fontsize=24
    )

    plt.axis("off")

    plt.tight_layout()

    plt.show()


if __name__ == "__main__":

    base_url = (
        "https://www.alps2alps.com"
    )

    max_depth = 2

    graph = crawl_website(
        base_url,
        max_depth
    )

    print()

    print(
        "URLs encontradas:",
        graph.number_of_nodes()
    )

    print(
        "Enlaces encontrados:",
        graph.number_of_edges()
    )

    visualize_graph(
        graph
    )
